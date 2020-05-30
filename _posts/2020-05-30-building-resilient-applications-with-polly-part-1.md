---
title: "Building Resilient .NET Core Applications With Polly - Part 1"
date: 2020-05-30
tags: [C#, .NET Core, Polly]
---

In this age of Service Oriented Architecture (SOA) where small microservices within a system communicate with each other using various protocols, typically over a network, it is important to note that there may be transient failures in some of the services for one reason or another. It's crucial that your system manages to recover from these impermanent failures when this happens. This is what resilience means. In this post we are going to talk about [Polly](http://www.thepollyproject.org/) -- an open source .NET library that provides resilience and transient-fault handling capabilities and how you can use it in your .NET Core applications. We are going to focus on the retry capabilities of Polly in this post.

## The Project

The sample project for this series, which is hosted on [GitHub](https://github.com/vince-nyanga/PollyExample), comprises of two .NET Core web API projects -- the Api and the Client. I am not going to add all the code in this post for brevity, instead, I'm going to summarise and show snippets that I think are important.

The Api project provides a RESTful API that the Client project consumes. Requests to the Api are not always successful by design for the purposes of demonstration purposes. Here is the `WeatherForecastController` inside the Api project:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.AspNetCore.Mvc;

namespace Api.Controllers
{
    [ApiController]
    [Route("api/weather")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        [HttpGet]
        public ActionResult<IEnumerable<WeatherForecast>> Get()
        {
            var rng = new Random();
            var randInt = rng.Next(2);

            if (randInt % 2 == 0)
            {

                return Ok(
                    Enumerable.Range(1, 5).Select(index => new WeatherForecast
                    {
                        Date = DateTime.Now.AddDays(index),
                        TemperatureC = rng.Next(-20, 55),
                        Summary = Summaries[rng.Next(Summaries.Length)]
                    })
                    .ToArray()
                );
            }

            return NotFound();
        }
    }
}
```

As you can see, calls to this service are successful 2 out of 3 times on average. Now let's turn our focus to the Client project.

The Client project is a web API application that depends on the Api project to get weather forecasts. It makes use of an interface -- `IWeatherService`:

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Client.Interfaces
{
    public interface IWeatherService
    {
        Task<IEnumerable<WeatherForecast>> GetWeatherForecast();
    }
}
```

The interface above is implemented by the `HttpWeatherService` that makes REST calls to the Api project to fetch the weather:

```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Client.Interfaces;
using Newtonsoft.Json;

namespace Client.Services
{
    public class HttpWeatherService : IWeatherService
    {
        private readonly HttpClient _client;

        public HttpWeatherService(HttpClient client)
        {
            _client = client;
        }

        public async Task<IEnumerable<WeatherForecast>> GetWeatherForecast()
        {
            var response = await _client.GetAsync("/api/weather");
            if (response.IsSuccessStatusCode)
            {
                var stringContent = await response.Content.ReadAsStringAsync();

                return JsonConvert.DeserializeObject<IEnumerable<WeatherForecast>>(stringContent);
            }
            else
            {
                return null;
            }
        }
    }
}
```

The `IWeatherService` interface is then injected into the `WeatherForecastController` in the Client project:

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Client.Interfaces;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

namespace Client.Controllers
{
    [ApiController]
    [Route("/")]
    public class WeatherForecastController : ControllerBase
    {

        private readonly ILogger<WeatherForecastController> _logger;
        private readonly IWeatherService _service;

        public WeatherForecastController(ILogger<WeatherForecastController> logger, IWeatherService service)
        {
            _logger = logger;
            _service = service;
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<WeatherForecast>>> Get()
        {
            _logger.LogInformation("Fetching weather from API");

            var forecast = await _service.GetWeatherForecast();
            if (forecast != null)
            {
                _logger.LogInformation("Weather successfully fetched");
                return Ok(forecast);
            }
            else
            {
                _logger.LogInformation("Failed to fetch weather");
                return NotFound();
            }
        }
    }
}
```

If we run both projects, after registering the `IWeatherService` interface in the service collection, we will find that we get a successful request every 2/3 times on average. This is a simulated transient fault that we are going to handle using Polly.

First, we need to add Polly to our Client project. Install these NuGet packages:

```
dotnet add package Polly
dotnet add package Microsoft.Extensions.Http.Polly
```

Let's go to the `Startup.cs` file inside the Client project and add the following code:

```csharp
using System;
using System.Net.Http;
using Client.Interfaces;
using Client.Services;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Polly;
using Polly.Extensions.Http;
using Serilog;

namespace Client
{
    public class Startup
    {
        // ...

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            services.AddHttpClient<IWeatherService, HttpWeatherService>(client =>
            {
                client.BaseAddress = new Uri("https://localhost:5001/");
            })
                .AddPolicyHandler(GetRetryPolicy()); // Add retry policy
        }

        private static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
        {
            Random jitterer = new Random();
            return HttpPolicyExtensions
                 .HandleTransientHttpError()
                 .OrResult(res => !res.IsSuccessStatusCode)
                 .WaitAndRetryAsync(
                      2,
                     retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
                                                           + TimeSpan.FromMilliseconds(jitterer.Next(0, 1000)),
                     onRetry: (response, span, retryCount, context) =>
                     {
                         Log.Information("Retry count: {RetryCount}", retryCount);
                     });

        }

        // ...
    }
}
```

Inside the `ConfigureService` method we add our weather service to the service collection using the `AddHttpClient` extension method. We then add a Polly retry policy to that `HttpClient`. Let's now talk about the retry policy in detail.

## Retry Policy

The retry policy allows callers to retry operations in the expectation that many faults are transient and may self-correct. If, after a configured number of retries, we still don't have a successful result, we can return an error to the user. For more information on the retry policy checkout the [wiki](https://github.com/App-vNext/Polly/wiki/Retry) for the Polly project on GitHub

This is what we are doing in the example above. Since our Api is successful only 2/3 times we don't want to return a `404: Not Found` to our users as soon as a call fails. Instead, we try again up to 2 times hoping that the initial failure was short-lived. If we still get a failure after 2 retries, we can return the error to the user or handle it in some other way. Below is a flow chart that shows how the retry policy works, courtesy of the Polly documentation:

<figure>
<img src="{{ site.baseurl }}/images/polly/retry.png" alt="Retry policy">
<figcaption>How the retry policy works. Courtesy: https://github.com/App-vNext/Polly/wiki/Retry </figcaption>
</figure>

### Retry Strategies

It is unwise for us to retry immediately when we get a failure as the upstream system may not have recovered from the fault which may overload the system. A well-known strategy called `exponential backoff` allows retries to be made initially quickly, but then at progressively longer intervals: for example, after 2, 4, 8, 15, then 30 seconds. This gives the upstream system a chance to recover and avoid overloading it.

The exponential backoff strategy on its own may not be suffient in high throughput systems as it may still overload the system. Instead a random jitter -- an extra delay may be added to avoid all clients sending calls simultaneously. This is what we have done in the example above. For more information checkout the [wiki](https://github.com/App-vNext/Polly/wiki/Retry-with-jitter).

## Conclusion

In this post we introduced Polly, an open source .NET resilience and transient-fault-handling library. We also discussed the retry policy and how is can be used to recover from transient upstream faults. This is the first post in a series where we will discuss various policies that Polly provides and how they can be used to make your system even more resilient. You may download the full source code for this post on [GitHub](https://github.com/vince-nyanga/PollyExample). Once again, thank you for taking your time to read and stay safe if you're reading this during the 2020 Covid-19 pandemic.

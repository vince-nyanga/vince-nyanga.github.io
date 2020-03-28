---
title: "Building an identity server that supports OAuth 2.0 and OpenID Connect with ASP.NET Core and IdentityServer4 - Part 2"
date: 2020-03-22
tags: [C#, ASP.NET Core]
---

In the [previous post]({{ site.baseurl}}/identity-server-part-1) we built an identity server that supports OAuth 2.0 and OpenID Connect protocols using the `IdentityServer4` framework. We also registered one API (`dummy_api`) that our identity server is going to protect, and one client (`console_app`) that will connect to the API. See the [Configuration.cs](https://github.com/vince-nyanga/IdentityServerTutorial/blob/master/Server/Configuration.cs) that we created in the previous post. In this post we are going to implement the protected API and the console app client. Let's get started.

## REST API

Let's start by creating our REST API that we registed in our identity server. It is going to be a simple API with just one protected endpoint. Create an empty ASP.NET Core web application named `Api` and install the `Microsoft.AspNetCore.Authentication.JwtBearer` NuGet package. We will use this package to validate the JWT token sent by the connected client. This is how it's going to work:

- The client gets a JWT access token from the identity server (see next section).
- The client then sends a request to the API with the token inside the `Authorization` header.
- The API then receives and validates the token before it grants access to its secure endpoints.

Now let's go to the `Startup.cs` class and add some configuration to our API:

```csharp
// Startup.cs

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace Api
{
    public class Startup
    {


        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            services.AddAuthentication("Bearer")
                .AddJwtBearer("Bearer", config =>
                {
                    // Address of your identity server
                    config.Authority = "https://localhost:44349/";

                    // Unique ID of the API as registered in the identity server
                    config.Audience = "dummy_api";
                });
        }


        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseAuthentication();
            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}

```

With our basic configuration done let's turn our attention to our simple API endpoint. Add `Controllers/ApiController.cs` and add the following code:

```csharp
// Controllers/ApiController.cs

using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace Api.Controllers
{
    public class ApiController : Controller
    {
        [Route("/secret")]
        [Authorize] // protect the endpoint
        public IActionResult Index()
        {
            return new JsonResult(new
            {
                secret = "I love ASP.NET Core"
            });
        }
    }
}

```

We have added a simple endpoint to our REST API that is protected. Now let's add a client that will connect to this API.

## Console Client

Create an ASP.NET Core console application called `ConsoleClient` and install this NuGet package that contains some extension methods that we are going to need: `IdentityModel`. Now add the following code to the `Program.cs` class:

```csharp
// Program.cs

using IdentityModel.Client;
using System;
using System.Net.Http;
using System.Threading.Tasks;

namespace ConsoleClient
{
    class Program
    {

         static async Task Main()
        {
            var identityServerUrl = "https://localhost:44349/";
            var apiUrl = "https://localhost:44334";

            var serverClient = new HttpClient();


            // Get the discovery document from the identity server
            var discoverDocument = await serverClient.GetDiscoveryDocumentAsync(identityServerUrl);

            // Get Access token for console app client
            var tokenResponse = await serverClient.RequestClientCredentialsTokenAsync(
                new ClientCredentialsTokenRequest
                {
                    Address = discoverDocument.TokenEndpoint,
                    ClientId = "console_app",
                    ClientSecret = "console_app_secret",
                    Scope = "dummy_api"
                });

            if (tokenResponse.IsError)
            {
                Console.WriteLine($"Failed to get access token. {tokenResponse.Error}");
                return;
            }

            var apiClient = new HttpClient();
            // Add Bearer toke to request header
            apiClient.SetBearerToken(tokenResponse.AccessToken);

            var response = await apiClient.GetAsync($"{apiUrl}/secret");
            var content = await response.Content.ReadAsStringAsync();

            Console.WriteLine($"API response:\n{content}");

        }
    }
}

```

This is what the console app is doing:

1. Gets the discovery document (see previous post) from the identity server.
2. Gets the access token using the client ID and client secret as registered in the `Configuration.cs` class in the identity server project.
3. Makes a request to the API with the access token added as a bearer token inside the `Authorization` header.

If the access token sent to the API passes validation then our console app client will have access to the protected `/secret` endpoint. Now run all the three projects -- `Server`, `API` and the `ConsoleClient`. You should be able to see the secret message displayed on your console. That's it for today. You can get the entire source code on [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial).

## Conclusion

In this post we implemented a REST API that we registered in the identity server we created in the previous post. The API accepts and validates JWT tokens from connected clients to grant access to a secure endpoint. We also implemented a simple console application that we registered in our identity server. This application gets a JWT access token from the identity server and adds it as a bearer token inside the `Authorization` header of requests to the API. The API then validates the token and grants the console app access to its protected endpoint. In the next post we will add support for user authentication to our identity server. Once again, thank you so much for taking your time to read.

---
title: "Using WireMock.Net To Mock HTTP APIs In .NET"
excerpt: "In this post, I will show you how to use WireMock.Net to mock HTTP APIs in .NET."
date: 2023-06-10
tags: [.NET, C#, Testing]
---

In this post I am going to show you how you can use WireMock.NET to mock your HTTP APIs. When writing tests for classes that make external HTTP calls, it is desirable to mock out those external calls so you can focus on the behavior of the system under test. This is where WireMock.NET shines. Let's look at how to use it.

## Example

I am going to create a simple service that makes an external HTTP call to get weather forecast. Here is the service:

```csharp
internal sealed class WeatherService : IWeatherService
{
    private const string WeatherForecastRequestUri = "forecast";
    private readonly HttpClient _httpClient;

    public WeatherService(HttpClient httpClient) =>
        _httpClient = httpClient;

    public async ValueTask<IReadOnlyCollection<WeatherForecast>> GetForecastsAsync() =>
        await _httpClient.GetFromJsonAsync<IReadOnlyCollection<WeatherForecast>>(WeatherForecastRequestUri);
}
```

When testing this service, we do not want to make a call to the real weather API. Instead, we want to mock the HTTP call and return a response that we control. First, let's install the package using the dotnet CLI:

```bash
dotnet add package WireMock.Net
```

We can write tests for our service:

```csharp
public sealed class WeatherServiceTests : IDisposable
{
    private readonly WireMockServer _mockServer;
    private readonly WeatherService _sut;

    public WeatherServiceTests()
    {
        _mockServer = WireMockServer.Start();
        var httpClient = new HttpClient{BaseAddress = new Uri(_mockServer.Url!)};
        _sut = new WeatherService(httpClient);
    }

    [Fact]
    public async Task GetForecastAsync_ShouldReturnForecastAsync()
    {
        // given
        var forecast = new[] { CreateRandomForecast() };
        _mockServer.Given(
            Request.Create()
                .WithPath("/forecast")
                .UsingGet())
            .RespondWith(
                Response.Create()
                    .WithBodyAsJson(forecast));

        // when
        var result = await _sut.GetForecastsAsync();

        // then
        result.Should().BeEquivalentTo(forecast);
    }

    public void Dispose() =>
        _mockServer.Dispose();

    private WeatherForecast CreateRandomForecast() =>
        new Filler<WeatherForecast>().Create();
}
```

In the above test, we are creating a mock server and passing its URL to the `HttpClient` instance. We then create a mock response and return it when the `/forecast` endpoint is called. The test passes! This is a simple example that shows how to use WireMock.NET. It is only a tip of the iceberg -- there is so much you can do with this library. For more information, check out the [documentation](https://github.com/WireMock-Net/WireMock.Net/wiki).

## Conclusion

In this post, I showed you how to use WireMock.NET to mock HTTP APIs in .NET. WireMock.NET is a great tool for mocking HTTP APIs. It is easy to use and has a lot of features. You can use it for other use cases which require mocking external APIs besides unit tests or integration tests. I hope you found this post useful. If you have any questions or comments, please leave them in the comments section below.

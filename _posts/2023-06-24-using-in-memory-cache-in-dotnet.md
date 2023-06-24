---
title: "Using In-Memory Cache In .NET"
excerpt: "In this post, we'll look at how to use the in-memory cache in .NET."
date: 2023-06-24
tags: [.NET, C#, Caching, Performance]
---

Caching plays a crucial role in improving the performance and scalability of applications. By storing frequently accessed data in a temporary location, caching reduces the time required to connect with the data source and send data across the network. This technique is particularly effective for data that changes infrequently but takes time to generate, or is expensive to retrieve. However, it is important to note that applications should never solely rely on cached data and should always have a fallback mechanism in place.

In the .NET ecosystem, there are several types of caching mechanisms available. In this article, we will focus on in-memory caching, one of the simplest and most commonly used caching techniques in ASP.NET Core applications. We will explore the basics of caching, the advantages of in-memory caching, and how to implement it effectively in .NET Core.

## What is Caching?

Caching is the process of storing frequently accessed data in a temporary storage, known as a cache. This technique improves performance by avoiding unnecessary hits to your database or external service and reducing the time complexity involved in fetching data. When a request for data is made, the application first checks the cache. If the data is found in the cache, it can be retrieved quickly without the need to access the original source. However, if the data is not present in the cache or has become stale, the application falls back to fetching it from the data source.

## Types of Cache

In the .NET Core ecosystem, two primary types of caching are supported: in-memory caching and distributed caching.

**1. In-Memory Caching:** In-memory caching stores data in the memory of the application server. It is best suited for small applications deployed on a single server. The data is stored in the cache and can be accessed rapidly, resulting in improved performance and responsiveness.

**2. Distributed Caching:** Distributed caching involves storing data on an external service that multiple application servers can share. This type of caching is necessary when you have multiple instances of your service running. Popular solutions for distributed caching include Redis and other third-party mechanisms.

For the purpose of this article, we will focus on in-memory caching, which is widely used and offers a straightforward implementation in .NET Core.

## Benefits of In-Memory Caching

In-memory caching offers several advantages for .NET Core applications:

- **Rapid Data Retrieval:** With in-memory caching, data can be fetched quickly from the cache, leading to improved performance and reduced response times. This is especially beneficial for frequently accessed data that does not change frequently.
- **Enhanced Application Performance:** By reducing the number of database hits, in-memory caching lightens the load on services and improves overall application performance. This is particularly useful in scenarios where the data source is expensive to access or generate.

- **Cost Savings:** By minimizing the need for frequent database queries, in-memory caching can save costs associated with data retrieval and processing. This is especially relevant in cloud-based environments where data transfer costs can be significant.

## Implementing In-Memory Caching in ASP.NET Core

To implement in-memory caching in ASP.NET Core, we can leverage the `IMemoryCache` interface, which represents a cache stored in memory. This interface is part of the `Microsoft.Extensions.Caching.Memory` package, which is recommended for ASP.NET Core applications due to its seamless integration with the framework.

To begin using in-memory caching, follow these steps:

- Install the `Microsoft.Extensions.Caching.Memory` package using NuGet.

- Configure the caching service by adding it to your service collection:

```csharp
services.AddMemoryCache();
```

- Inject the `IMemoryCache` interface into your class or controller where you want to use caching.

The example below shows how to use in-memory caching to get weather from a weather service.

```csharp
internal sealed class WeatherService : IWeatherService
{
    private const string CacheKey = "WeatherForecasts";
    private static readonly SemaphoreSlim Semaphore = new(1,1);

    private readonly IWeatherBroker _broker;
    private readonly IMemoryCache _cache;
    private readonly ILogger<WeatherService> _logger;

    public WeatherService(IWeatherBroker broker, IMemoryCache cache, ILogger<WeatherService> logger)
    {
        _broker = broker;
        _cache = cache;
        _logger = logger;
    }

    public async ValueTask<IEnumerable<WeatherForecast>> GetAsync()
    {
        if (TryGetCachedForecasts(out var forecasts))
        {
            _logger.LogInformation("Returning cached forecasts");
            return forecasts;
        }

        try
        {
            await Semaphore.WaitAsync();

            if (TryGetCachedForecasts(out forecasts))
            {
                _logger.LogInformation("Returning cached forecasts");
                return forecasts;
            }

            _logger.LogInformation("Fetching forecasts from broker");

            forecasts = (await _broker.GetForecastsAsync()).ToArray();

            var cacheEntryOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromSeconds(5))
                .SetAbsoluteExpiration(TimeSpan.FromSeconds(10));

            _cache.Set(CacheKey, forecasts, cacheEntryOptions);

            return forecasts;
        }
        finally
        {
            Semaphore.Release();
        }
    }

    private bool TryGetCachedForecasts(out IEnumerable<WeatherForecast> forecasts) =>
        _cache.TryGetValue(CacheKey, out forecasts);
}
```

In the example above, we first check if the data is present in the cache. If it is, we return the cached data. Otherwise, we fetch the data from the weather broker and store it in the cache. We also set a sliding expiration of 5 seconds and an absolute expiration of 10 seconds to ensure that the data is refreshed periodically. This ensures that the data remains up to date and reflects any changes that may have occurred in the original source. We also use a semaphore to ensure that only one thread can update the cache at a time.

## Best Practices for Caching in .NET

While caching can greatly enhance application performance, it is important to follow certain best practices to ensure its effectiveness:

- **Always Have a Fallback Mechanism:** Caching should never be the sole source of data. Applications must have a fallback mechanism in place to fetch data from the original source if it is not available in the cache. This ensures that the application remains functional even if the cache becomes invalidated or unavailable.

- **Limit Cache Growth:** Caches utilize memory resources, and it is important to limit cache growth to prevent excessive memory usage. Define a proper cache eviction policy, such as a maximum number of items or a time-based expiration, to manage the cache size effectively.

- **Cache Data, Not External Input:** Avoid inserting external input directly into the cache to prevent security vulnerabilities such as cache poisoning or injection attacks. Always sanitize and validate input before using it to retrieve or store data in the cache.

- **Use a Combination of Absolute and Sliding Expiration:** A good caching strategy involves using a combination of absolute and sliding expiration. This ensures that the cache entry expires by an absolute time irrespective of whether it's still active or not, thereby preventing cache entries from becoming stale.

- **Test and Monitor Cache Performance:** Proper testing and monitoring are essential to ensure the effectiveness and efficiency of the caching mechanism. Test different cache configurations and monitor cache hit rates, memory usage, and response times to identify any performance bottlenecks or areas for optimization.

By adhering to these best practices, you can maximize the benefits of caching in your .NET applications and create highly performant and scalable solutions.

## Conclusion

Caching is a powerful technique for improving the performance and scalability of .NET applications. By leveraging in-memory caching, developers can significantly reduce the time required to fetch data from the original source, resulting in faster response times and improved user experience. Implementing in-memory caching in ASP.NET Core is straightforward, thanks to the `IMemoryCache` interface and the integration with the framework.

However, it is important to remember that caching should never be relied upon as the sole source of data. Applications should always have a fallback mechanism in place and regularly refresh the cache to ensure data consistency and reliability. By following best practices and monitoring cache performance, developers can harness the full potential of caching in .NET, leading to highly optimized and efficient applications.

I hope you enjoyed this article about caching in .NET. If you have any questions or feedback, please free to leave a comment below or reach out to me on [Twitter](https://twitter.com/honesdev). Until next time, happy coding!s

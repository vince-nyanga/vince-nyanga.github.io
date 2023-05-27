---
title: "Using The Rate Limiting Middleware In ASP.NET Core"
excerpt: "In this post, we'll look at how to use the rate limiting middleware in ASP.NET Core."
date: 2023-05-27
tags: [.NET, Rate limiting, ASP.NET Core, Middleware]
---

In the [previous post]({{ site.baseurl}}/rate-limiting-in-dotnet), I introduced the rate limiting API that was added in .NET 7 as well as the four algorithms that comes with the library. I also showed a simple example of how to use it. In this post, we'll look at how to add rate limiting to an ASP.NET Core application using the rate limiting middleware.

## Using The Rate Limiting Middleware

Since I have already spoken in detail about how rate limiting works, I won't go into that again. Instead, let's look at how to use the rate limiting middleware in ASP.NET Core.

First, you need to configure the rate limiting options.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.OnRejected = (context, cancellationToken) =>
    {
        if (context.Lease.TryGetMetadata(MetadataName.RetryAfter, out var retryAfter))
        {
            context.HttpContext.Response.Headers.RetryAfter =
                ((int) retryAfter.TotalMilliseconds).ToString(NumberFormatInfo.InvariantInfo);
        }

        context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        context.HttpContext.Response.WriteAsync(
            text: "Too many requests. Please try again later.",
            cancellationToken: cancellationToken);

        return new ValueTask();
    };
    options.AddFixedWindowLimiter(
        policyName: Constants.RateLimitPolicyName,
        fixedWindowRateLimiterOptions =>
        {
            fixedWindowRateLimiterOptions.PermitLimit = 1;
            fixedWindowRateLimiterOptions.Window = TimeSpan.FromSeconds(3);
        });
});
```

The above code adds a fixed window rate limiter with a limit of 1 request per 3 seconds. It also sets the `OnRejected` callback which is invoked when a request is rejected. In the callback, we set the `Retry-After` header and the status code to `429`. We also write a message to the response body. The client can use the `Retry-After` header to determine when to retry the request. All the algorithms that we discussed in the previous post can be configured in the same way.

Next, we need to add the rate limiting middleware to the pipeline.

```csharp
app.UseRateLimiter();
```

Now let's add an endpoint that we can use to test the rate limiting.

```csharp
var group = routes.MapGroup("weather")
    .WithTags("weather")
    .WithOpenApi()
    .RequireRateLimiting(Constants.RateLimitPolicyName);
```

If you run this application, your first request will succeed. If you make another one within 3 seconds, it will fail with a `429` status code. The response will also contain the `Retry-After` header.

## Rate Limiting Policies

We can achieve the same result by defining a rate limiting policy. The class should implement the `IRateLimiterPolicy<TPartitionKey>` interface.

```csharp
internal class FixedWindowRateLimitingPolicy : IRateLimiterPolicy<string>
{
    private readonly ILogger<FixedWindowRateLimitingPolicy> _logger;

    public FixedWindowRateLimitingPolicy(ILogger<FixedWindowRateLimitingPolicy> logger)
    {
        _logger = logger;
    }

    public RateLimitPartition<string> GetPartition(HttpContext httpContext)
    {
        return RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: string.Empty,
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 1,
                Window = TimeSpan.FromSeconds(3)
            });
    }

    public Func<OnRejectedContext, CancellationToken, ValueTask> OnRejected => (context, cancellationToken) =>
    {
        if (context.Lease.TryGetMetadata(MetadataName.RetryAfter, out var retryAfter))
        {
            _logger.LogDebug(
                message: "Too many requests. Retry after {RetryAfter} ms",
                args: retryAfter.TotalMilliseconds.ToString(NumberFormatInfo.InvariantInfo));

            context.HttpContext.Response.Headers.RetryAfter =
                ((int)retryAfter.TotalMilliseconds).ToString(NumberFormatInfo.InvariantInfo);
        }

        context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        context.HttpContext.Response.WriteAsync(
            text: "Too many requests. Please try again later.",
            cancellationToken: cancellationToken);

        return new ValueTask();
    };
}
```

We are creating a rate limit partition with a fixed window rate limiter. A rate limit partition will allow us to set rate limiting options for a specific partition, for instance, a user. Our policy will be applied on a per-user basis in that case instead of applying it globally which won't make sense in the real world.

Then we can add the policy during configuration.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy<string, FixedWindowRateLimitingPolicy>(Constants.RateLimitPolicyName);
});
```

Besides the cleaner code, the advantage of defining your policy is that you can inject dependencies into it and use them as you see fit.

## Conclusion

In this post, we looked at how to use the rate limiting middleware in ASP.NET Core. We also saw how to define a rate limiting policy. I hope you learned something. The source code for this post is available on [GitHub](https://github.com/vince-nyanga/RateLimitingSample) if you want to take a look at it. If you have any questions or suggestions, feel free to leave a comment below. Till next time, happy learning!

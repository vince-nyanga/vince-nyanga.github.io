---
title: "Rate Limiting in .NET"
excerpt: "In this post I will be talking about rate limiting in .NET."
date: 2023-05-20
tags: [.NET, Rate limiting]
---

Rate limiting is a technique used to control the rate at which a user can access a resource. It is used to prevent abuse of the resource which may lead to denial of service or high costs, if you are using a service that charges per request. In this post I will be talking about rate limiting in .NET. When .NET 7 was released in November 2022, it introduced a new package [System.Threading.RateLimiting](https://www.nuget.org/packages/System.Threading.RateLimiting). This package provides the a foundation for rate limiting and provides a few rate limiting algorithms out of the box. In this post I will talk about these algorithms as well as create a simple console application to demonstrate how to use them. Let's get started.

## The RateLimiter API

Before we talk about the algorithms, let's look at the API. The `RateLimiter` class is the base class in the `System.Threading.RateLimiting` namespace. It is an abstract class that looks like this:

```csharp
public abstract class RateLimiter : IAsyncDisposable, IDisposable
{
    public RateLimitLease AttemptAcquire(int permitCount = 1)
    {
        if (permitCount < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(permitCount));
        }

        return AttemptAcquireCore(permitCount);
    }

    protected abstract RateLimitLease AttemptAcquireCore(int permitCount);

    public ValueTask<RateLimitLease> AcquireAsync(int permitCount = 1, CancellationToken cancellationToken = default)
    {
        if (permitCount < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(permitCount));
        }

        if (cancellationToken.IsCancellationRequested)
        {
            return new ValueTask<RateLimitLease>(Task.FromCanceled<RateLimitLease>(cancellationToken));
        }

        return AcquireAsyncCore(permitCount, cancellationToken);
    }

    protected abstract ValueTask<RateLimitLease> AcquireAsyncCore(int permitCount, CancellationToken cancellationToken);

    // Other methods omitted for brevity
}

```

At the core of the rate limiting API is the concept of a lease represented by the `RateLimitLease` class. The calling code needs to acquire a lease first before it can access the resource that is being rate limited. The `RateLimiter` class has two methods that can be used to acquire a lease:

- `AttemptAcquire` - This method attempts to acquire a lease synchronously. It delegates how the lease is acquired to the `AttemptAcquireCore` method which is implemented by the derived classes. If the lease is not available, it will return a failed lease indicating that the calling code does not have access to the resource at this time. If the lease is available, it will return a successful lease.
- `AcquireAsync` - This method attempts to acquire a lease asynchronously. It allows for the calling code to wait until the permits/tokens are available instead of immediately returning a failed lease. It delegates how the lease is acquired to the `AcquireAsyncCore` method which is implemented by the derived classes.

There is another abstract class in the API called `ReplenishingRateLimiter` which inherits from the `RateLimiter` class. It provides an API to replenish tokens after a specified time interval. Most of the algorithms I am going to talk about inherit from this class. The `ReplenishingRateLimiter` class looks like this:

```csharp
public abstract class ReplenishingRateLimiter : RateLimiter
{
    public abstract TimeSpan ReplenishmentPeriod { get; }

    public abstract bool IsAutoReplenishing { get; }

    public abstract bool TryReplenish();
}
```

The `ReplenishmentPeriod` property specified how often the tokens are replenished. The `IsAutoReplenishing` property specifies whether the tokens are replenished automatically or not. If it is set to `false`, the calling code needs to call the `TryReplenish` method to replenish the tokens. The `TryReplenish` method returns `true` if the tokens were replenished successfully and `false` if the tokens do not need to be replenished or when `IsAutoReplenishing` is set to `true`.

## The Algorithms

The `System.Threading.RateLimiting` namespace provides four rate limiting algorithms out of the box. Let's look at each of them.

### 1. ConcurrencyLimiter

This algorithm inherits from the `RateLimiter` class. It limits the number of consumers that can access a resource concurrently. It specifies the maximum number of concurrent consumers that can access the resource, say 5. If there are already 5 consumers accessing the resource, the next consumer will have to wait in the queue which also has a maximum size. If the queue is full, the consumer will be denied access to the resource. This assumes that the consumer used the `AcquireAsync` method which is asynchronous. If it uses the `AttemptAcquire` and there's no available tokens, access will be denied immediately. Once a consumer is done accessing the resource, a token is released and the next consumer can acquire it and access the resource.

### 2. TokenBucketRateLimiter

This algorithm inherits from `ReplenishingRateLimiter`. As the name suggests, this algorithm makes use of a bucket of tokens. When a consumer comes in, it picks up a token from the bucket and gains access to the resource. If the token bucket is empty, the consumer will have to wait in the queue, if it's not already full, otherwise access will be denied. A specified number of tokens are added back to the bucket after a specified time interval which is specified by the `ReplenishmentPeriod` property. However, the number of tokens in the bucket cannot exceed the maximum number of tokens the bucket is set to hold. If the number of tokens in the bucket is already equal to the maximum number of tokens, the tokens will not be replenished.

To give an example, suppose we have a bucket that can hold 5 tokens and we replenish 1 token every minute. When a consumer comes in, it picks up a token from the bucket and we have 4 tokens left. If 4 more consumers come in within the minute, the bucket will be empty. The next consumer will have to wait in the queue. After a minute, 1 token will be added back to the bucket and the next consumer can pick it up and access the resource.

### 3. FixedWindowRateLimiter

This algorithm also inherits from `ReplenishingRateLimiter`. It specifies a maximum number of consumers that are allowed to access a resource within a specified time interval, say 5 consumers in 30 seconds. When the 30 seconds has elapsed, we move to the next window in which the number of tokens available is reset allowing the next 5 consumers to access the resource. If the number of consumers that come in within the 30 seconds is greater than 5, the next consumer will have to wait in the queue, if it's not already full, otherwise access will be denied.

### 4. SlidingWindowRateLimiter

This algorithm is similar to the `FixedWindowRateLimiter` algorithm, but adds smaller windows (segments) within the large window. If the window is 1 minute and we have 2 segments, each segment will be 30 seconds long. The algorithm maintains a pointer to the current segment. When the segment time elapses, the pointer moves to the next segment. Once a segment slides out of the window (expires), the number of tokens used in that segment are recycled back into the window. The remaining tokens in the window are calculated using this formula: $$ remaining = available - acquired + recycled $$ Let's look at an example to understand this better.

Suppose we have a 1 minute window which has a limit of 10 requests and has 2 segments. In the first segment, 2 requests are made. This means there are now 8 tokens available to the next segment. In segment 2, 5 requests are made which means that we only have 3 tokens available. We then slide the window by 30 seconds. Segment 1 is expired and the token used in that segment are recycled back into the window. No requests were made in segment 3 so the remaining tokens in the window are 5 ($$ 3 - 0 + 2 $$). The table below shows the state of the window after each segment. I have added more segments to further demonstrate the algorithm. If a request comes in in segment 5, which doesn't have any available tokens, it will be added to the queue until the window slides again at which point 10 tokens will be recycled back into the window from segment 4.

| Segment | Available | Acquired | Recycled | Remaining |
| ------- | --------- | -------- | -------- | --------- |
| 1       | 10        | 2        | 0        | 8         |
| 2       | 8         | 5        | 0        | 3         |
| 3       | 3         | 0        | 2        | 5         |
| 4       | 5         | 10       | 5        | 0         |
| 5       | 0         | 0        | 0        | 0         |
| 6       | 0         | 0        | 10       | 10        |

## Example

I have created a simple calculator that uses rate limiting. Let's call it the lazy calculator. Here is the code for the calculator:

```csharp
internal sealed class Calculator
{
    private readonly RateLimiter _rateLimiter;

    public Calculator(RateLimiter rateLimiter)
    {
        _rateLimiter = rateLimiter;
    }

    public async ValueTask<int> AddAsync(int a, int b)
    {
        Log.Debug("Request to add {A} and {B}", a, b);

        using (var lease = await _rateLimiter.AcquireAsync(permitCount: 1))
        {
            if (lease.IsAcquired)
            {
                Log.Debug("Acquired lease for {A} and {B}", a, b);
                return a + b;
            }

            Log.Warning("Failed to acquire lease for {A} and {B}", a, b);

            var retryMessage = lease.TryGetMetadata(MetadataName.RetryAfter, out var retryAfterValue)
                ? $"Retry after {retryAfterValue.TotalMilliseconds} ms"
                : "Retry later";

            throw new InvalidOperationException($"{a} + {b} cannot be calculated at this time. {retryMessage}");
        }
    }
}
```

The calculator has a method called `AddAsync` which adds two numbers. The method uses the `AcquireAsync` method to acquire a lease asynchronously. If the lease is acquired, the method returns the sum of the two numbers. If the lease is not acquired, the method throws an exception. The exception message contains the `RetryAfter` value which is the time after which the request can be retried. The calculator uses the `MetadataName.RetryAfter` metadata name to get the `RetryAfter` value from the lease. If the calling code implements a retry mechanism, it can use this value to determine when to retry the request.

Now lets look at how we can use the calculator. We will use the `SlidingWindowRateLimiter` algorithm to limit the number of requests to 5 in 30 seconds. Here is the code:

```csharp
var fixedWindowRateLimiter = new FixedWindowRateLimiter(new FixedWindowRateLimiterOptions
{
    PermitLimit = 1,
    Window = TimeSpan.FromSeconds(5),
    QueueLimit = 2,
    QueueProcessingOrder = QueueProcessingOrder.OldestFirst
});

var calculator = new Calculator(fixedWindowRateLimiter);

var tasks = Enumerable.Range(0, 5)
    .Select(_ => AddRandomNumbersAsync(calculator));

await Task.WhenAll(tasks);

static async Task AddRandomNumbersAsync(Calculator calculator)
{
    try
    {
        var a = new Random().Next(1, 10);
        var b = new Random().Next(1, 10);

        var result = await calculator.AddAsync(a, b);

        Log.Information("Result for {A} and {B} is {Result}", a, b, result);
    }
    catch (InvalidOperationException)
    {
        Log.Error("Calculator is busy");
    }
}

```

The code above creates a `FixedWindowRateLimiter` with a window of 5 seconds and a limit of 1 request. It then creates a calculator and calls the `AddAsync` method 5 times. The `AddAsync` method adds two random numbers. The code above will produce the following output:

```powershell
[07:37:48 DBG] Request to add 5 and 9
[07:37:48 DBG] Acquired lease for 5 and 9
[07:37:48 INF] Result for 5 and 9 is 14
[07:37:48 DBG] Request to add 3 and 2 # this is queued
[07:37:48 DBG] Request to add 2 and 3 # this is queued
[07:37:48 DBG] Request to add 9 and 3 # queue is full so this is rejected
[07:37:48 WRN] Failed to acquire lease for 9 and 3
[07:37:48 ERR] Calculator is busy
[07:37:48 DBG] Request to add 7 and 7 # queue is full so this is rejected
[07:37:48 WRN] Failed to acquire lease for 7 and 7
[07:37:48 ERR] Calculator is busy
[07:37:53 DBG] Acquired lease for 3 and 2 # this is dequeued 5sec later
[07:37:53 INF] Result for 3 and 2 is 5
[07:37:58 DBG] Acquired lease for 2 and 3 # this is dequeued 5sec later
[07:37:58 INF] Result for 2 and 3 is 5
```

As you can see, the first request is successful. The second and third requests are queued. The fourth and fifth requests are rejected because the queue is full. The second and third requests are dequeued 5 seconds after the previous successful request (when we go into the next window) and are successful.

## Conclusion

In this post, I spoke about the rate limiting API that was added in .NET 7. I also explained the 4 rate limiting algorithms that come with the API. I then showed how to use the API to rate limit requests to a calculator. The code for this post can be found on [GitHub](https://github.com/vince-nyanga/LazyCalculator). In the next post, I will show you how to add rate limiting to an ASP.NET Core web API. If you have any questions, comments or suggestions, don't hesitate to leave them in the comment section below. Until next time, keep learning 😊.

---
title: "Building Resilient .NET Core Applications With Polly's Circuit Breaker Policy"
date: 2020-06-06
tags: [C#, .NET Core, Polly]
---

In the [previous post]({{ site.baseurl }}/building-resilient-applications-with-polly-part-1/) we introduced Polly, a .NET resilience and transient-fault-handling library. We spoke about the retry policy that can be used to help your application properly handle transient failures. In this post we are going to take a look at another policy that Polly provides -- the Circuit Breaker policy. Let's get into it.

## The Circuit Breaker Policy

The retry policy discussed in the previous post is very effective if the failure in the subsystem was short-lived. If the subsystem is not available at all then no amount of retries will solve the problem. This is where the circuit breaker policy comes in handy.

The circuit breaker policy detects the level of faults in calls placed through it, and prevents calls when a configurable fault threshold is exceeded. It works just like the circuit breaker in your house that manages the flow of electricity to various parts of your house. If a given threshold of faults from a subsystem is reached the circuit will open thereby preventing calls to the subsystem.

The circuit breaker policy can be wrapped around the retry policy if you want to achieve more resilience in your system. We will discuss wrapping policies in a later post.

### How Does It Work

The circuit breaker policy works like a state machine with 3 states -- **Open**, **Closed**, and **Half Open** state. Initially, the circuit is in the _Closed_ state and when the threshold of failures is reached it goes to the _Open_ state for a configurable amount of time. All calls sent through it during this time will throw a `BrokenCircuitException`. After the configured amount of time in the _Open_ state has elapsed, the circuit breaker moves to the _Half Open_ state where it will allow only **ONE** call through to test if the subsystem being called is now responding. If the call succeeds then the circuit will close to allow calls to go through, otherwise it will go back to the _Open_ state.

For more indepth information check out the [Polly wiki](https://github.com/App-vNext/Polly/wiki/Circuit-Breaker).

## Example

In our example web application (see code on [GitHub](https://github.com/vince-nyanga/PollyExample/tree/circuit-breaker)) we can add the circuit breaker policy when registering our service:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddHttpClient<IWeatherService, HttpWeatherService>(client =>
    {
        client.BaseAddress = new Uri(Configuration.GetValue<string>("ApiUrl"));
    })
        .AddPolicyHandler(GetCircuitBreakerPolicy());
}

private IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 2,
            durationOfBreak: TimeSpan.FromSeconds(5),
            onBreak: OnBreak,
            onReset: OnReset,
            onHalfOpen: OnHalfOpen);
        }

private void OnHalfOpen()
{
    Log.Information("Circuit is Half Open");
}

private void OnReset()
{
    Log.Information("Circuit reset");
}
```

In the example above, the circuit will be broken (go to _Open_ state) after 2 failures and it will remain open for 5 seconds. During this time all calls will fail immediately without ever reaching the system being called. After 5 seconds, it will move to _Half Open_ state allowing one call to go through and test if the subsystem is now responsive so it can move to _Closed_ or _Open_ state depending on the result.

### Advanced Circuit Breaker

So far we've been talking about the basic circuit breaker policy. There is an [advanced](https://github.com/App-vNext/Polly/wiki/Advanced-Circuit-Breaker) version that gives you more control of how the circuit will behave. This circuit breaker reacts to the proportion of failures, for instance, the circuit will open if 50% of calls through it result in a handled failure. It measures that proportion over a configured duration, say 20 seconds, and will act if a minimum throughput was reached during the interval. Here's an example of how you can set up an advanced circuit breaker:

```csharp
 private IAsyncPolicy<HttpResponseMessage> GetAdvancedCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .AdvancedCircuitBreakerAsync(
            failureThreshold: 0.5, // open if 50% of calls fail
            samplingDuration: TimeSpan.FromSeconds(30), // measure failures in a 30 sec interval
            minimumThroughput: 10, // act only if at least 10 calls were made during the interval
            durationOfBreak: TimeSpan.FromSeconds(5),
            onBreak: OnBreak,
            onReset: OnReset,
            onHalfOpen: OnHalfOpen);
        }
```

## Conclusion

In this post we spoke about how Polly's circuit breaker policy can be used to further add resilience to an application. The source code for this post can be found on [GitHub](https://github.com/vince-nyanga/PollyExample/tree/circuit-breaker). Ensure you are in the `circuit-breaker` branch when you run the code. Once again, thanks so much for taking your time to read.

### Further Reading:

- The [Circuit breaker](https://github.com/App-vNext/Polly/wiki/Circuit-Breaker) policy on Polly's wiki
- The [Advanced circuit breaker](https://github.com/App-vNext/Polly/wiki/Advanced-Circuit-Breaker) policy on Polly's wiki
- [Twilio](https://www.twilio.com/blog/using-polly-circuit-breakers-resilient-net-web-service-consumers) blog on the circuit breaker policy

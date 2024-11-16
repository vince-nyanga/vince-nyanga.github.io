---
title: "Building A Message-Based System Using MassTransit - Introduction To MassTransit"
excerpt: "In this post, I will introduce MassTransit and explain how it can be used to build a scalable message-based system."
date: 2024-10-15
tags: [MassTransit, Messaging, Microservices]
---

This is the second post in the series `"Building A Message-Based System Using MassTransit"`. Here is an overview of what we will cover in this series:

- [Part 1]({{ site.baseurl}}/masstransit-prologue): Prologue: Introducing HonesRemit
- **Part 2**: Introduction to MassTransit _(this post)_
- **Part 3**: Add MassTransit to HonesRemit _(coming soon)_
- **Part 4**: Understanding the MassTransit message pipeline _(coming soon)_
- **Part 5**: Implementing outbox pattern with MassTransit _(coming soon)_
- **Part 6**: Implementing saga pattern with MassTransit _(coming soon)_
- **Part 7**: Writing tests for MassTransit _(coming soon)_
- **Part 8**: MassTransit vs NServiceBus _(coming soon)_

In the [previous post]({{ site.baseurl}}/masstransit-prologue), I introduced HonesRemit, a fictitious money transfer service that allows users to send money to their loved ones across the globe. I also showed a basic implementation of the API. In this post, we will take a look at MassTransit, a powerful open-source framework for building message-based applications in .NET. By the end of this post, you will have a good understanding of what MassTransit is and how it can be used to build scalable and reliable distributed systems.

## What is MassTransit?

MassTransit is a free, open-source distributed application framework for .NET. It provides a lightweight message bus for building distributed applications using the .NET framework. MassTransit enables developers to build scalable and reliable systems by providing abstractions for common messaging patterns such as publish/subscribe, request/response, and message routing. Just like [NServiceBus]({{ site.baseurl}}/nservicebus-introduction), MassTransit provides an abstraction over the underlying messaging infrastructure, allowing developers to focus on building business logic without worrying about the intricacies of the messaging transport. It supports multiple messaging transports, including RabbitMQ, Azure Service Bus, and Amazon SQS, among others. Let's now dive into some of the core concepts of MassTransit.

### Message

Just like in any messaging system, messages are the core building blocks of communication in MassTransit. A message is a simple data structure that represents a unit of information that needs to be sent from one component to another. MassTransit messages must be reference types and should be serializable. Here is an example of a simple message class:

```csharp
public class OrderSubmitted
{
    public Guid OrderId { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Amount { get; set; }
}
```

There are two types of messages in MassTransit: commands and events. Commands are messages that represent an action that needs to be performed, while events are messages that represent something that has happened. In the example above, `OrderSubmitted` is an event that indicates that an order has been submitted. There is no difference in how commands and events are defined in MassTransit; they are both regular message classes -- no marker interfaces or base classes are required.

### Endpoint

An endpoint refers to a destination or address where messages are sent or received. It can be thought of as a communication channel between services or components within a message-based architecture. An endpoint is typically associated with a queue or topic where messages are published, consumed, or both. We can think of endpoints as mailboxes where messages are delivered.

In the `OrderSubmitted` example above, there can be multiple endpoints that have subscribed to this event. When an order is submitted, MassTransit will deliver the `OrderSubmitted` message to all the subscribed endpoints. This is obviously a simplification of how MassTransit works, but it gives you an idea of what an endpoint is.

### Consumer

A consumer is a component that process messages received by an endpoint. It is responsible for handling messages and executing the necessary business logic. There are many consumer types: message consumers, batch consumers, sagas, saga state machines, among others. Multiple consumers can be connected to an endpoint.

#### Message Consumer

This is the most common consumer type. It is a class that consumers one or more message types. For each message type, the class implements `IConsumer<TMessage>` :

```cs
public interface IConsumer<in TMessage> : IConsumer
	where TMessage : class
{
	Task Consume(ConsumeContext<TMessage> context);
}
```

Using the `OrderSubmitted` message example, here is how you would implement a consumer for it:

```csharp
public class OrderSubmittedConsumer : IConsumer<OrderSubmitted>
{
    public async Task Consume(ConsumeContext<OrderSubmitted> context)
    {
        var order = context.Message;
        // do something with the order
    }
}
```

While the `Consume` method is executing, the message is unavailable to other receive endpoints. If the `Task` completes successfully, the message is acknowledged and removed from the queue. If it faults (fails) or cancelled, the consumer instance is released and an exception is thrown which, if it doesn't trigger a retry, the default pipeline will move the message to an error queue.

To add a consumer and automatically configure a receive endpoint for the consumer, call one of the `AddConsumer` methods and call `ConfigureEndpoints`:

```cs
services.AddMassTransit(x =>
{
    // add a single consumer
    x.AddConsumer<OrderSubmittedConsumer>();

    var entryAssembly = Assembly.GetEntryAssembly();

    // add all consumers in the specified assembly
    x.AddConsumers(entryAssembly);

    x.UsingInMemory((context, cfg) =>
    {
        cfg.ConfigureEndpoints(context);
    });
});
```

It is recommended to automatically configure receive endpoints by using the `AddConsumers(assembly)` method instead of having to add each consumer manually using the `AddConsumer<T>()` method. The `AddConsumers(assembly)` method will scan the specified assembly for all consumer types and add them to the container. It ensures your configuration is cleaner and less prone to someone forgetting to add a consumer.

You can provide more configuration to a consumer by using a consumer definition. Consumer definitions are used to specify the behaviour of consumers so that they can be automatically configured. They can be explicitly added by the `AddConsumer` method or auto-discovered using any of the `AddConsumers` methods. Here is an example of a consumer definition.

```cs
public class OrderSubmittedConsumerDefinition : ConsumerDefinition<OrderSubmittedConsumer>
{
	public SubmitOrderConsumerDefinition()
	{
		// override the default endpoint name, if you want
		EndpointName = "honesremit-order-submitted";

		// limit the number of messages consumed concurrently
		// this applies to the consumer only, not the endpoint
		ConcurrentMessageLimit = 4;
	}

	protected override void ConfigureConsumer(
        IReceiveEndpointConfigurator endpointConfigurator,
         IConsumerConfigurator<DiscoveryPingConsumer> consumerConfigurator)
	{
        // retry configure retry policy
		endpointConfigurator.UseMessageRetry(r => r.Interval(5, 1000));
	}
}
```

There are other ways of configuring consumers but I personally prefer using consumer definitions as they provide a clean way of configuring consumers and endpoints.

## Conclusion

In this post, we introduced MassTransit and explained some of its core concepts, including messages, endpoints, and consumers. We also showed how to create a simple message consumer in MassTransit. In the next post, we will add MassTransit to HonesRemit and start building our message-based system. Stay tuned for more exciting content!

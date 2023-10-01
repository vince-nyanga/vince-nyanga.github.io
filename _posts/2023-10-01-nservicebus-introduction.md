---
title: "An introduction to NServiceBus"
excerpt: "In this post, I will introduce NServiceBus, a very powerful library for building distributed systems in dotnet."
date: 2023-10-01
header:
  overlay_image: /images/nservicebus/nservicebus.png
tags: [NServiceBus, Messaging, Microservices]
---

Microservices and distributed architectures are riding a crest of a wave the past couple of years. They provide a lot of benefits and, if your usecase fits, you definitely need to consider them. In this post, I will introduce NServiceBus, a very powerful library for building distributed systems in dotnet.

At the end of this few minutes read, you will have an understanding of what NServiceBus is, and its main concepts. Let's get started.

## What is NServiceBus

NServiceBus is an implementation of the Enterprise Service Bus (ESB) pattern in dotnet. It enables developers to build decoupled services that can communicate with one another by sending and receiving messages through a service bus. An enterprise service bus is implemented using a message queue where services enqueue messages and interested parties listen for those messages and act on them. There are multiple queuing products one can use to implement this architecture. Examples include RabbitMQ, Azure Service Bus and AWS SQS, among others. Figure 1 below shows how an enterprise service bus works conceptually. The services interact with each other via the message queue.

<figure>
<img src="{{ site.baseurl }}/images/nservicebus/servicebus.png" alt="enterprise service bus">
<figcaption>Figure 1: Enterprise service bus</figcaption>
</figure>

NServiceBus is itself does not provide queuing infrastructure. It provides an abstraction above the queuing infrastructure, allowing you to easily change the underlying infrastructure without changing much in your code. This allows you to focus on implementing your business rules without worrying much about the infrastructure.

<figure>
<img src="{{ site.baseurl }}/images/nservicebus/nservicebus.png" alt="NServiceBus">
<figcaption>Figure 2: Enterprise service bus implementation using NServiceBus</figcaption>
</figure>

In Figure 2 above, each service is now interacting with the underlying queue infrastructure using NServiceBus.

## NServiceBus Concepts

There are a few concepts that you will need to understand before using NServiceBus. This post focuses on those concepts and subsequent posts will dive deeper into how NServiceBus works.

### Endpoint

At the core of NServiceBus is an endpoint. An endpoint is a fundamental unit of scale in NServiceBus. It is a logical component that communicates with other endpoints using [messages](#message). Imagine you have a system that comprises of sales and shipping microservices. In order for these microservices to communicate with each other using NServiceBus, an NServiceBus endpoint needs to be defined for each microservice. In Figure 2 above, we have four NServiceBus endpoints, one for each service. An endpoint should have a unique identifying name and is deployed to an environment.

This is how you define your endpoint:

```cs
var endpointConfiguration = new EndpointConfiguration("serviceA");
```

The `EndpointConfiguration` class provides APIs for configuring your endpoint, including setting the [transport](#transport) used, message routing, message serialization and configuring persistence for sagas.

### Endpoint Instance

An endpoint instance, also known as a physical endpoint, is the physical deployment of an [endpoint](#endpoint). It is the one that is physically connected to the backing queue infrastructure. Each endpoint must have at least one endpoint instance and you can scale an endpoint by deploying multiple endpoint instances. The endpoint instance is created from an endpoint configuration:

```cs
var endpointInstance = await Endpoint.Start(endpointConfiguration);
```

### Transport

NServiceBus contains an abstraction for underlying queuing technologies. An implementation of that abstraction for a given queuing technology is known as a transport. As I have already mentioned above, NServiceBus allows you to change the underlying queuing infrastructure without needing to make significant changes to your code. There are many transports supported by NServiceBus. To view the entire list, check out the [documentation](https://docs.particular.net/transports/#supported-transports).

### Message

As I have mentioned above, [endpoints](#endpoint) communicate with each other using messages. NServiceBus messages represent data contracts between endpoints. There are three types of messages in NServiceBus -- commands, events and replies.

#### Command

A command, like the name suggests, is a message that requests an action to be performed. It must have a logical owner, that is, an endpoint from which it originates, and must be sent to a single logical endpoint. For instance, a `CreateOrder` command must be sent to the sales endpoint and nowhere else. NServiceBus provides a routing mechanisms to configure the destination for a given command. If destination is not configured, you will get a runtime exception when you try to send the command.

Another characteristic of a command is that it must be acted upon. You cannot have a command that you send and don't expect the receiving service to handle it. In addition, a command should contain all the information needed for the receiving endpoint to act on it.

Here is an example of how you send a command from an endpoint instance:

```cs
var command = new MyCommand();
await endpointInstance.Send(command);
```

#### Event

An event on the other hand, is a message that notifies that an action has already occurred. Just like a command, an event also has a logical owner. However, it can be published to multiple endpoints -- it does not need to be directed to one endpoint. Basically, whoever is interested in an event will be able to handle it. Events don't need to contain all the information like commands do, you may decide to create _thin_ events which only contain details of the event that has occurred, for instance, an order ID.

Here's an example of how you publish an event:

```cs
var myEvent = new MyEvent();
await endpointInstance.Publish(myEvent);
```

#### Reply

Replies are neither commands nor events. These are messages that are sent back to the source endpoint (or saga) when a message has been handled. They are very useful when implementing the async request/reply pattern. Replies are sent in the context of handling a message, that is, inside a [message handler](#message-handler). Here's an example of how you can send a reply to the endpoint that sent a message:

```cs
// code omitted for brevity
public async Task Handle(CreateOrder message, IMessageHandlerContext context)
{
    // create and persist order
    var reply = new CreateOrderReply(status: Status.Successful);
    await context.Reply(reply);
}
```

### Message handler

A message handler is a class NServiceBus uses to handle messages received from the queue. Each [endpoint](#endpoint) must contain message handlers to handle all the messages it is interested in. [Commands](#command) can only have one handler while [events](#event) may have more than one handler, or none at all. NServiceBus handlers must implement the `IHandleMessages<TMessage>` interface. This is how a message handler is defined:

```cs
class CreateOrderHandler : IHandleMessages<CreateOrder>
{
    public async Task Handle(CreateOrder message, IMessageHandlerContext context)
    {
        // create and persist order
    }
}
```

## Conclusion

In this post I introduced NServiceBus and its core concepts. In the next post I will showcase how you can use it to build decoupled systems, bringing to life all the concepts I introduced in this post. If you have a comment, question or suggestion, feel free to leave a comment below.

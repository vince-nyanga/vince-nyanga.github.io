---
title: "Using Azure Service Bus In .NET"
excerpt: "In this post, I will show how to use Azure Service Bus in .NET."
date: 2023-07-15
tags: [Azure, Messaging, Service Bus]
---

Azure Service Bus, as one of the most powerful and flexible messaging services, has become a cornerstone in the creation of highly reliable and scalable applications. This article aims to provide an in-depth understanding of Azure Service Bus, its implementation using .NET and C#, and how it can be leveraged to facilitate asynchronous messaging patterns between applications.

## What Is Azure Service Bus

Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics. Service Bus is used to decouple applications and services from each other, providing the following benefits:

- Load-balancing work across competing workers
- Safely routing and transferring data and control across service and application boundaries
- Coordinating transactional work that requires a high-degree of reliability

It acts as a message broker, offering message queues and publish-subscribe topics within a namespace. It provides several benefits, such as load balancing, safe routing, and transaction coordination. Data transfer between applications and services is accomplished using messages. These messages, decorated with metadata, can carry any kind of information, including structured data encoded in common formats like JSON, XML, Apache Avro, or plain text.

Azure Service Bus supports various messaging scenarios:

- **Messaging:** Facilitates the transfer of business data like sales orders, purchase orders, etc.

- **Decoupling Applications:** Helps enhance application reliability and scalability.

- **Load Balancing:** Allows multiple consumers to read from a queue simultaneously.

- **Topics and Subscriptions:** Enables one-to-many relationships between publishers and subscribers.

- **Transactions:** Supports several operations within the scope of an atomic transaction.

- **Message Sessions:** Provides a mechanism for grouping related messages.

## Azure Service Bus Concepts

When working with Azure Service Bus, there are several concepts to be aware of:

- **Queues:** Messages are sent to and received from queues, which store the messages until the receiving application is ready to process them. Queues are useful in point-to-point communication scenarios. Only one consumer can receive and process a message from a queue.

- **Topics:** Messages can also be sent and received via topics, which are useful in publish/subscribe scenarios. Unlike queues, topics can have multiple subscriptions, each of which can have multiple consumers. Each subscription receives a copy of every message sent to the topic.

- **Subscriptions:** Subscriptions allow consumers to receive a copy of each message sent to a topic. They can have filters and actions to select and modify messages.

- **Namespaces:** A namespace is a container for all messaging components. It can have multiple queues and topics and often serves as an application container. If you want to use Service Bus, you must first create a namespace.

## Using Azure Service Bus In .NET

There is a client library for Azure Service Bus that can be used to send and receive messages. The library is available as a NuGet package, and it can be installed using the following command:

```bash
dotnet add package Azure.Messaging.ServiceBus
```

### Creating A Service Bus Client

Once you have installed the package you can create a Service Bus client using the following code:

```csharp
const string connectionString = "<connection-string>";
var client = new ServiceBusClient(connectionString);
```

If you are going to host your service on Azure, I highly recommend using [Managed Identity](https://honesdev.com/using-azure-managed-identity/) to authenticate with Azure Service Bus.

### Sending Messages

In order to send messages, you need to use an instance of the `ServiceBusSender` class. You can create an instance of this class using the `CreateSender` method of the `ServiceBusClient` class:

```csharp
var sender = client.CreateSender("<queue-name>");
```

The `ServiceBusSender` class provides many methods for sending messages, including sending a batch of messages, sending delayed messages etc. For more information, check out the [documentation](https://learn.microsoft.com/en-us/dotnet/api/azure.messaging.servicebus.servicebussender?view=azure-dotnet). The following code shows how to send a single message:

```csharp
var message = new ServiceBusMessage("Hello World!");
await sender.SendMessageAsync(message);
```

> Note: The ServiceBusSender is safe to cache and use for the lifetime of an application or until the ServiceBusClient that it was created by is disposed. Caching the sender is recommended when the application is publishing messages regularly or semi-regularly.

### Receiving Messages

In order to receive messages, you need to use an instance of the `ServiceBusReceiver` class. You can create an instance of this class using the `CreateReceiver` method of the `ServiceBusClient` class:

```csharp
var receiver = client.CreateReceiver("<queue-name>");
```

To receive messages, you can use the `ReceiveMessageAsync` method of the `ServiceBusReceiver` class:

```csharp
var message = await receiver.ReceiveMessageAsync();
var body = message.Body.ToString();
```

#### Handling Messages

Once you are done processing a message, there are options you have available to handle the message:

- **Complete:** This will remove the message from the queue or subscription. You call the `CompleteMessageAsync` method of the `ServiceBusReceiver` class to complete a message.
- **Abandon:** This will abandon the message and make it available to be received again. You call the `AbandonMessageAsync` method of the `ServiceBusReceiver` class to abandon a message.
- **Defer:** Deferring a message will prevent it from being received again using the standard receive methods. Instead, you need to use the `ReceiveDeferredMessageAsync` to receive deferred messages.
- **Dead-letter:** Dead lettering a message moves it to a sub-queue of the original queue, preventing the message from being received again. You need a receiver scoped to the dead letter queue to receive messages from the dead letter queue.

#### Service Bus Processor

The `ServiceBusProcessor` class provides an abstraction around a set of ServiceBusReceiver that allows using an event based model for processing received `ServiceBusReceivedMessage`. It is constructed by calling `CreateProcessor(String, ServiceBusProcessorOptions)` on the service bus client. Here is an example of how to use it:

```csharp
var processor = client.CreateProcessor("<queue-name>");
processor.ProcessMessageAsync += MessageHandler;
processor.ProcessErrorAsync += ErrorHandler;
await processor.StartProcessingAsync();

static async Task MessageHandler(ProcessMessageEventArgs args)
{
    var body = args.Message.Body.ToString();
    await args.CompleteMessageAsync(args.Message);
}

static Task ErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine(args.Exception.ToString());
    return Task.CompletedTask;
}
```

When using the `ServiceBusProcessor`, you won't need to manually fetch messages from the queue. The processor will automatically fetch messages and call the `ProcessMessageAsync` event handler. The `ProcessErrorAsync` event handler will be called if there is an error while processing the message.

## Conclusion

In this post, we looked at how to use Azure Service Bus in .NET. We looked at how to create a Service Bus client, how to send and receive messages, and how to use the Service Bus processor. I hope you found this post useful. If you have any questions or comments, please leave a comment below.

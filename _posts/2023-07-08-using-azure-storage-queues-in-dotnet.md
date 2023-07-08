---
title: "Using Azure Storage Queues In .NET"
excerpt: "In this post, I will show how to use Azure Storage Queues in .NET."
date: 2023-07-08
tags: [Azure, Messaging, Storage Queues]
---

Azure Queue Storage is a cloud-based service that allows you to store a vast number of messages that can be accessed via authenticated HTTP or HTTPS calls from any location globally. Each message in the queue has a maximum size of 64 KB, while the queue itself can store an enormous number of messages -- the capacity of the storage account. In this post, I will show you how to use Azure Storage Queues in .NET. I will show you how to create a queue, add messages to it, and read messages from it.

## Prerequisites

Before you begin, you need to have an Azure subscription. If you don't have one, you can create a [free account](https://azure.microsoft.com/en-us/free/). If you don't want to create an account, that's okay too. You can use Azurite, which is a local storage emulator that simulates the Azure Storage services for testing purposes. You can download it from [here](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite). If you are going to use the real Azure Queue Storage, you need to create a Storage Account. You can find how to do it [here](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal).

Once you have that in place, you need to install the Azure Storage Queue NuGet package. You can do it by running the following command in the Package Manager Console.

```bash
dotnet add package Azure.Storage.Queues
```

## Creating a Queue

To create a queue, you need to create an instance of the `QueueClient` class. You need to pass the connection string or use managed identity, and the name of the queue to the constructor. The connection string can be found in the Azure Portal. You can find it in the Access Keys section of the Storage Account. You can optionally pass in an instance of the `QueueClientOptions` class to the constructor. The `QueueClientOptions` class allows you to configure the retry policy, message encoding and other options. Here is how you can create a queue in C#.

```csharp
const string connectionString = "your_connection_string";
const string queueName = "your_queue_name";

var queueClient = new QueueClient(connectionString, queueName);

await queueClient.CreateIfNotExistsAsync();
```

The code snippet above is pretty straight forward. You create an instance of the `QueueClient` class and call the `CreateIfNotExistsAsync` method. If the queue doesn't exist, it will be created. If it exists, nothing will happen.

## Adding Messages to a Queue

Azure Storage Queues support two types of messages: string messages and binary messages. To add a message to a queue, you need to call the `SendMessageAsync` method on the `QueueClient` class. The method accepts a `string` or `BinaryData` as a message body. You can optionally pass in a time-to-live and a visibility timeout. The time-to-live is the time that the message will be stored in the queue. The visibility timeout is the time that the message will be invisible to other consumers after it has been dequeued. Here is how you can add a message to a queue in C#.

```csharp
var message = "Hello, World!";
var timeToLive = TimeSpan.FromMinutes(5);
var visibilityTimeout = TimeSpan.FromSeconds(30);
await queueClient.SendMessageAsync(message, timeToLive, visibilityTimeout);
```

If your message is a complex object, you will need to serialize it before sending it to the queue.

> **_NOTE:_** You need to ensure that your message encoding is the same between the sender and the receiver.

## Reading Messages from a Queue

There are two options for reading messages from a queue: peeking and dequeuing. When you peek a message, it will remain in the queue. When you dequeue a message, it will be removed from the queue and other processes can access it. To peek a message, you need to call the `PeekMessagesAsync` method on the `QueueClient` class. The method accepts the number of messages to peek. Here is how you can peek a message from a queue in C#.

```csharp
var peekedMessages = await queueClient.PeekMessagesAsync(1);
var peekedMessage = peekedMessages.Value.FirstOrDefault();
```

To dequeue a message, you need to call the `ReceiveMessagesAsync` method on the `QueueClient` class. The method accepts the number of messages to dequeue. Here is how you can dequeue a message from a queue in C#.

```csharp
var dequeuedMessages = await queueClient.ReceiveMessagesAsync(1);
var dequeuedMessage = dequeuedMessages.Value.FirstOrDefault();
```

When you call the `ReceiveMessageAsync` method, the message(s) will become invisible to other consumers for the visibility timeout period. After the visibility timeout period has elapsed, the message(s) will become visible again. If you are done processing the message and don't want it to be visible for other consumers agains, you need to call the `DeleteMessageAsync` method on the `QueueClient` class to remove it from the queue. The method accepts the message ID and the pop receipt. Here is how you can delete a message from a queue in C#.

```csharp
await queueClient.DeleteMessageAsync(dequeuedMessage.MessageId, dequeuedMessage.PopReceipt);
```

## Demo Application

I have a demo application that shows how to use Azure Storage Queues in .NET. You can find it on [GitHub](https://github.com/vince-nyanga/storage-queue-sample). It consists for two console applications, one for sending messages and the other for receiving them. It uses [Coravel](https://docs.coravel.net/) to schedule the jobs. Feel free to clone it and play around with it.

## Conclusion

In this post, I showed how to use Azure Storage Queues in .NET. I showed how to create a queue, add messages to it, and read messages from it. I hope you have found this post useful. Storage queues can be very useful when you want to build a decoupled message driven system that doesn't cost an arm and a leg. It is important to note that storage queues are very simple and don't have advanced features such as message ordering or transactions. I hope you have learned something and thanks so much for reading.f

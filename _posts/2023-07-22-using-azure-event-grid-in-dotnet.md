---
title: "Using Azure Event Grid In .NET"
excerpt: "In this post, I will show how to use Azure Event Grid in .NET."
date: 2023-07-22
tags: [Azure, Messaging, Event Grid]
---

Azure Event Grid is a remarkable solution for developers working with event-based architectures. It plays a pivotal role in managing the routing of events from any source to any destination, for any application. This service can handle events from Azure services and custom events which can be published directly to the service. These events can then be filtered and sent to various recipients, such as built-in handlers or custom webhooks. In this article, we will delve deeper into the Azure Event Grid and its .NET client library.

## Azure Event Grid Concepts

Azure Event Grid's functionality can be understood through these concepts:

- **Event:** Describes what happened.
- **Event source:** Specifies where the event took place.
- **Topic:** The endpoint where publishers send events.
- **Event subscription:** The endpoint or built-in mechanism to route events, sometimes to more than one handler. Subscriptions also help handlers to intelligently filter incoming events.
- **Event handler:** The application or service responding to the event.

## Event Schemas

Event Grid supports two schemas for encoding events -- event grid schema and cloud events v1.0 schema. When a topic or domain is created, you need to specify the schema that will be used when publishing events.

### Event Grid schema

This is the default schema selected if you don’t specify a schema. This is how the Event Grid Schema looks like:

```json
{
  "topic": "string",
  "subject": "string",
  "id": "string",
  "eventType": "string",
  "eventTime": "string",
  "data": {},
  "dataVersion": "string",
  "metadataVersion": "string"
}
```

### CloudEvents v1.0 schema

Another option is to use the CloudEvents v1.0 schema. CloudEvents is a Cloud Native Computing Foundation project which produces a specification for describing event data in a common way. Here is how the schema looks like:

```json
{
  "specversion": "1.0",
  "type": "string",
  "source": "string",
  "id": "string",
  "time": "string",
  "subject": "string",
  "data": {}
}
```

## Advantages of Azure Event Grid

The Azure Event Grid comes with tangible benefits:

- It supports native event handling mechanisms in the Azure cloud application, enabling swift connections between data sources and event handlers.
- It supports both built-in and custom events.
- It provides intelligent routing with filters and standardizes an event schema.
- It is a highly reliable service with 24 hours retry.
- It can support millions of events per second.
- It greatly enhances serverless, ops automation, and integration work.

## Using Azure Event Grid In .NET

There is a client library available to .NET developers. The library provides the following functionality:

- Publish events to the Event Grid service using the Event Grid Event, Cloud Event 1.0, or custom schemas
- Consume events that have been delivered to event handlers
- Generate SAS tokens to authenticate the client publishing events to Azure Event Grid topics

To use it in your application, you need to install if from NuGet:

```bash
dotnet add package Azure.Messaging.EventGrid
```

### Publishing Messages

The library provides the `EventGridPublisherClient` class that allows you to publish events to a topic or domain. First you need to create a new instance of the client:

```csharp
var client = new EventGridPublisherClient("<topic-endpoint>", new AzureKeyCredential("<access-key>"));
```

The example above uses an access key to authenticate the client. If you are going to host your application in Azure, I highly recommend that you authenticate using a managed identity. The `EventGridPublisherClient` also accepts a set of configuring options through `EventGridPublisherClientOptions`. For example, you can specify a custom serializer that will be used to serialize the event data to JSON.

Once you have authenticated your client, you can start publishing events. Regardless of what schema your topic or domain is configured to use, `EventGridPublisherClient` will be used to publish events to it. Use the `SendEvent` or `SendEventAsync` method for publishing single events, or `SendEvents`/`SendEventsAsync` if you want to publish multiple events:

```csharp
// Sending an Event Grid event
var eventGridEvent = new EventGridEvent(
    "This is the event data",
    "ExampleEventSubject",
    "Example.EventType",
    "1.0");

await client.SendEventAsync(eventGridEvent);

// Sending a cloud event
var cloudEvent = new CloudEvent(
    "/cloudevents/example/source",
    "Example.EventType",
    "This is the event data");

await client.SendEventAsync(cloudEvent);
```

## Receiving Events

There are several different Azure services that act as event handlers. These include Azure Functions, Logic Apps or your own custom webhooks.

> **Note:** when using webhooks for handling your events, Event Grid requires you to prove ownership of your webhook endpoint before it starts delivering events to that endpoint.

Once events are delivered to the event handler, parse the JSON payload into a list of events.

```csharp
// Using Event Grid event
EventGridEvent[] events = EventGridEvent.Parse(jsonPayloadSampleOne);
foreach (var egEvent in events)
{
  var data = egEvent.GetData();
  // do something with the data.
}

// using cloud event
foreach (CloudEvent cloudEvent in cloudEvents)
{
    switch (cloudEvent.Type)
    {
        case "MyApp.Users.Added":
        var userAdded = cloudEvent.GetData<UserAddedEvent>();
        Console.WriteLine(user.Email);
        break;
        case "MyApp.Vouchers.Redeemed":
        var voucherRedeemed = await cloudEvent.GetDataAsync<VoucherRedeemedEvent>();
        Console.WriteLine(voucherRedeemed.Code);
        break;
    }
}
```

## Conclusion

In this post I showed you how you can use Azure Event Grid in your .NET applications. I hope you have learned something. If you have a question, comment or suggestion, please feel free to leave it below. Thanks so much for taking your time to read.

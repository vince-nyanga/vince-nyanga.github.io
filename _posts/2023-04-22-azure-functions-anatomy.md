---
title: "Azure Functions 101: Anatomy of Azure Functions"
excerpt: "In this post, I will be describing the anatomy of Azure Functions."
date: 2023-04-22
tags: [Azure, Azure Functions, Serverless]
---

This is the second post in a series: [Azure Functions 101]({{ site.baseurl}}/series/azure-functions-101). In this series I will be covering the following topics:

- Part 1: [What Is Azure Functions]({{ site.baseurl}}/azure-functions-intro)
- Part 2: Anatomy of Azure Functions (this post)
- Part 3: Deploying Azure Function Resources Using Bicep and GitHub Actions (coming soon)
- Part 4: Creating Azure Functions using the Azure Functions Core Tools (coming soon)
- Part 5: Deploying changes to Azure Functions code using GitHub Actions (coming soon)

In the [previous article]({{ site.baseurl}}/azure-functions-intro), I introduced Azure Functions -- its use cases, benefits, and limitations. In this post I will be describing the anatomy of Azure Functions.

**Table of Contents**:

- [Azure Function App](#azure-function-app)
- [Runtime](#runtime)
- [Scale Controller](#scale-controller)
- [Application Settings](#application-settings)
- [Logging and Monitoring](#logging-and-monitoring)
- [Triggers](#triggers)
- [Function Code](#function-code)
- [Bindings](#bindings)
- [Conclusion](#conclusion)

## Azure Function App

This is a collection of one or more Azure Functions. You cannot deploy a single Azure Function in isolation. It must be deployed as part of a Function App. In other words, a Function App is the deployment unit for Azure Functions. All functions inside a Function App share the same configuration, such as the runtime version, the application settings, and the storage account.

## Runtime

The Functions runtime is responsible for running your code. The runtime includes logic on how to trigger, log, and manage function executions.

## Scale Controller

The scale controller is responsible for managing the number of instances of a function that are running. It monitors the rate of events that trigger the function, and adjusts the number of accordingly. It is responsible for ensuring that the function is able to handle the load that it is receiving. If you are running your Azure Functions in Docker containers, you'll only have the runtime and not the scale controller. You will have to manage the scaling of your functions yourself using tools such as KEDA.

The runtime and the scale controller are the two key components of the Azure Functions service. As a developer, you don't need to worry much about them. Of course you're are responsible for choosing the runtime version that you want to use, but the scale controller is managed by Azure Functions.

## Application Settings

Application settings are configuration values that a function can use at runtime. They can be used to store connection strings, API keys, environment variables, or any other values that the function needs to access during execution. Application settings can be stored securely in the Azure portal or in a `local.settings.json` file during development.

## Logging and Monitoring

Azure Functions provide built-in logging and monitoring capabilities that allow developers to diagnose issues and optimize performance. The logging system captures detailed information about the execution of a function, including input and output data, exceptions, and timing information. The monitoring system provides real-time metrics and alerts for functions, allowing developers to identify issues and respond quickly.

## Triggers

Triggers, as the name suggests, are the events that trigger the execution of an Azure Function. Each function should have only one trigger. Triggers have associated data, which is often provided as the payload of the function. There are several types of triggers available in Azure Functions. The following table lists some of the available triggers:

| Trigger                                                                                                                | Description                                                       |
| ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| [Blob](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=csharp)     | Triggered when a new blob is added to a blob storage container.   |
| [Cosmos DB](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-trigger?tabs=csharp) | Triggered when a new document is added to a Cosmos DB collection. |
| [Event Grid](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-grid?tabs=csharp)         | Triggered when an event is published to an Event Grid topic.      |
| [Event Hub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs-trigger?tabs=csharp)  | Triggered when a new message is added to an Event Hub.            |
| [HTTP](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook?tabs=csharp)             | Triggered when an HTTP request is received.                       |
| [IoT Hub](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-iot-trigger)                | Triggered when a new message is added to an IoT Hub.              |
| [Queue](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp)   | Triggered when a new message is added to a queue.                 |
| [Timer](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=csharp)                   | Triggered on a schedule.                                          |

Here is an example of a function that is triggered by an HTTP request:

![http trigger](/images/azure-function-trigger.svg)

The function trigger is defined by the `HttpTrigger` attribute. The trigger's associated data is provided as the `HttpRequest` parameter. The `HttpRequest` parameter is an object that contains information about the HTTP request, such as the request body, headers, and query string parameters.

## Function Code

This is the code that is executed when the function is triggered. The function code is written in any of the supported programming languages, such as C#, JavaScript, Python, and F#. It should be noted however, that once you have selected your runtime, all your functions need to be written in the language supported by that runtime. Azure Functions are designed to be lightweight and short-lived, so the function code should be optimized for performance and efficiency.

## Bindings

Bindings provide a way to declaratively connect other resources to the function. They are provided to the function as parameters. By using bindings, developers can write less code to interact with external systems, as the binding takes care of the details of connecting to and interacting with the external system. There two types of bindings: input and output bindings. Input bindings are used to read data from other resources, while output bindings are used to write data to other resources. The following table lists some of the available bindings:

| Binding                                                                                                          | Description                                                  |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | --- |
| [Blob](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob?tabs=csharp)       | Used to read and write data to a blob storage container.     |
| [Cosmos DB](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2?tabs=csharp)   | Used to read and write data to a Cosmos DB collection.       |
| [Event Grid](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-grid?tabs=csharp)   | Used to publish events to an Event Grid topic.               |
| [Event Hub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs?tabs=csharp)    | Used to read and write data to an Event Hub.                 |     |
| [IoT Hub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-iot?tabs=csharp)       | Used to read and write data to an IoT Hub.                   |
| [Queue](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue?tabs=csharp)     | Used to read and write data to a queue.                      |
| [Service Bus](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus?tabs=csharp) | Used to read and write data to a Service Bus queue or topic. |
| [SignalR](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-signalr-service?tabs=csharp) | Used to send messages to a SignalR Hub.                      |
| [Table](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-table?tabs=csharp)     | Used to read and write data to a table storage table.        |

Each binding has a corresponding attribute that is used to define the binding. The attribute takes in parameters that define how to connect to the external resource. For example, the `Blob` attribute takes in the path to the blob storage container as a parameter. I have enhanced the previous example to use an output binding to publish an event to an Event Grid topic:

![output binding](/images/azure-functions-bindings.svg)

The output binding is defined by the `EventGrid` attribute that takes in the topic endpoint and the topic key as parameters.

## Conclusion

In this post I provided an overview of the different components that make up an Azure Function, including the function trigger, function code, bindings, function host, application settings, logging and monitoring. I hope you found this post useful. If you have any questions or comments, please leave them below.

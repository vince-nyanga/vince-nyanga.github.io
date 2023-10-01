---
title: "Implementing the saga pattern in NServiceBus"
excerpt: "In this post, I will show you how you can implement the saga pattern in NServiceBus"
date: 2023-10-15
header:
  overlay_image: /images/nservicebus/nservicebus.png
tags: [NServiceBus, Messaging, Microservices, Saga]
---

A saga represents a high-level business process that comprises of several low-level requests, each of which updates data within a single service. These requests may take a long time to run and may span multiple microservices. A saga acts as an orchestrator for such requests, ensuring that all they are performed in a given sequence.

By the end of this three minute read, you will have basic understanding of how sagas work in NServiceBus as well has how you can implement specific actions such as sending replies to saga callers.

## Sagas in NServiceBus

NServiceBus has out-of-the-box support for sagas. To implement a saga, you need to create a class that inherits from `Saga<T>`, when `T` is the saga data class. The saga data class need to inherit from the `ContainSagaData` class. The saga data must contain a property that uniquely identifies the corresponding saga instance. You will see why this is important in the [correlation](#message-correlation) section. Most, if not all sagas have some state. Since sagas can run for a very long time, the state needs to be persisted somewhere where the saga will be able to rehydrate it when it needs it. The saga data class is the saga's state. This is how you configure persistence for you sagas:

```cs
endpointConfiguration.UsePersistence<PersistenceToUseGoesHere>();
```

## Starting a saga

The trigger to start a saga is the arrival of one or more specified message types. This is declared by adding `IAmStartedByMessages<TMessage>` to your saga class. When a message of the specified type is receive, a new saga instance is created, if there isn't one already that correlates to the message. In NServiceBus, a saga requires one message to start it.

## Adding behavior to a saga

You can add more behavior to a saga by implementing the `IHandleMessages<TMessage>` interface for other message types other than the one that starts the saga.

## Message correlation

When a message arrives from the queue, NServiceBus needs to know which saga instance will handle it. The `Saga<T>` class contains an abstract method `ConfigureHowToFindSaga` in which you will map properties of your messages to a property on the saga data. This is the reason why your saga data must have a unique property that will be used to correlate to an instance of the saga.

### Auto-correlation

There are instances where you don't need to manually configure how to find your saga for a given message. This is true when dealing with replies. A reply is a message that is sent by a message handler to the originator of the message when it is done handling the message. This is a classic request/reply implementation. In the case of sagas, all messages sent from a saga contain a header that can be used to identify the saga instance. Likewise, all replies sent back to the saga will contain the same correlation header. NServiceBus will use this header to identify the saga so there's no need to manually configure correlation inside the `ConfigureHowToFindSaga` method.

> **Note**:
> A limitation of this feature is that it doesn't support auto-correlation between sagas. If the request is handled by another saga, relevant message properties must be added and mapped to the requesting saga using the syntax described above.

## Notifying saga callers of status

Imagine this fictitious business process: an employee needs to be issues a company device. Before they can be issue that device, they need to get approval from their manager as well as the procurement manager. A saga can be used to implement this workflow. Now imagine the calling system, let's call it the device management system, needs to be updated of the status of the process so it can update it's UI or something. NServiceBus sagas are able to notify calling systems with ease. This is because the saga data contains the original system's endpoint address, as well as the original message ID. All you need to do is call the `ReplyToOriginator` message:

```cs
public async Task Handle(ManagerApprovalCommand message, IMessageHandlerContext context)
{
	// do something..
	var statusMessage = new StatusMessage
	{
		Id = "<some-id>",
        Status = "manager approved"
	};

	ReplyToOriginator(context, statusMessage);
}
```

## Completing a saga

To end a saga, call the `MarkAsComplete()` method. This tells the saga infrastructure that the instance is no longer needed and can be cleaned up.

## Conclusion

In this post I introduces NServiceBus sagas and how you can use them to orchestrate complex business processes.

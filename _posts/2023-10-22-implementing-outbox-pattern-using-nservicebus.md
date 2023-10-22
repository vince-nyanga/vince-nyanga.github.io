---
title: "Implementing the outbox pattern using NServiceBus"
excerpt: "In this post, I will show you how you can implement the outbox pattern using NServiceBus."
date: 2023-10-22
header:
  overlay_image: /images/nservicebus/nservicebus.png
tags: [NServiceBus, Messaging, Microservices, Outbox]
---

In a message based system, it is a common occurrence that messages need to be sent after updating transaction data. For instance, once an order has been created, we might need to publish an event so that other services may act on the event. The challenge however, is that we will be dealing with two infrastructure components -- our transactional database and the message broker. We therefore need to atomically update the database as well as send a message to the broker to avoid inconsistencies. We don't want a situation where the we update our database and fail to send the message or vice versa.

This is where the outbox pattern comes in. The solution is for the service that sends the message to first store the message in the **same** database as part of the transaction that updates the business entities. A separate process then sends the messages to the message broker.

At the end of this few minute read, you will have a deep understanding of how NServiceBus implements the outbox pattern as well as design considerations you need to be aware of when using the feature.

## Outbox pattern in NServiceBus

NServiceBus supports the outbox pattern out-of-the-box. It simulates an atomic transaction, distributed across both the data store used for business data and the message queue used for messaging. The outbox feature also guarantees that each message is processed once and only once. The implementation of the outbox pattern in NServiceBus involves two separate phases:

### Phase 1

1. Receive the incoming message from the queue.
   - Do not acknowledge receipt of the message yet, so that if it fails, the message will be delivered again.
2. Check the outbox storage in the database to see if the incoming message has already been processed (_deduplication_). The message ID is used for deduplication.
   - If the message has already been processed, skip to **step 7**.
   - If the message has not yet been processed, continue to **step 3**.
3. Begin a transaction in the database.
4. Execute the message handler for the incoming message.
   - Any outgoing messages are not immediately sent.
5. Store any outgoing messages in the outbox storage in the database.
6. Commit the transaction in the database
   - This is the operation that ensures consistency between messaging and database operations.
7. Go to phase 2.

### Phase 2

1. Check if the outgoing messages have already been sent.
   - If they have been sent, the incoming message is a duplicate, skip to **step 4**.
   - If they have not been sent yet, continue to **step 2**.
2. Send the outgoing messages to the queue.
   - If processing fails at this point, duplicate messages may be sent to the queue. However, since the duplicate messages will have the same `MessageId`, they will be deduplicated by the outbox feature in the endpoint that receives them.
3. Update outbox storage to show that the outgoing messages have been sent.
4. Acknowledge (ACK) receipt of the incoming message so that it's removed from the queue and will not be delivered again.

## Enabling the outbox feature on an endpoint

Before you start using the outbox feature, you first need to enable it on your endpoint. This is done by calling the `EnableOutbox` method on the endpoint configuration instance:

```cs
endpointConfiguration.EnableOutbox();
```

Your endpoint transport (the underlying queue) must also be explicitly set to `ReceiveOnly` mode. This ensures that messages dispatched via the outbox cannot be rolled back by the transport after the changes have already been persisted to the outbox storage.

## Persistence

The outbox feature requires persistent storage in order to perform deduplication and store outgoing downstream messages. You need to configure your endpoint to use a certain persistence:

```cs
endpointConfiguration.UsePersistence<PersistenceToUseGoesHere>();
```

## Design considerations

Here are a few things you need to consider when using the outbox pattern with NServiceBus:

- **Storage requirements**: A single outbox record, after all the transport operations have been dispatched requires less than 50 bytes of storage. These records need to stay in your database for a period of time for deduplication purposes. You can calculate how much storage you will need using the equation: $$ r = t \times d $$, where $$ r $$ is the total outbox records, $$ t $$ is the message throughput per second, and $$ d $$ is the deduplication per second.
- **Performance**: For best performance, outbox data should be stored in the same database as the business data.
- **Duplicate messages**: The outbox is expected to generate duplicate messages from time to time, especially if there is unreliable communication between the endpoint and the transport. This however, is not a big issue since there is built in deduplication so you are guaranteed to handle the message once.

## Conclusion

In this post, I explained in detail how the outbox pattern is implemented by the NServiceBus library. I also spoke about a few things you need to consider or be aware of when using the outbox pattern with NServiceBus.

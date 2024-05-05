---
title: "An Introduction To Event Sourcing"
excerpt: "In this post, I will introduce event sourcing, a different way to manage application state."
date: 2024-03-02
tags: [System Design, Event Sourcing]
---

Data is an integral part of most, if not all, software systems built the world over. As a result, its capturing (collection), storage, and analysis are among the most important things one needs to pay attention to, and optimise, in order to meet a system's functional and non-functional requirements. In most applications, the typical approach to handling data is for the application to maintain the current state of the data by updating it as users work with it. This state is persisted in some form of durable storage.

Let's take a look at a fictitious and overly simplified banking system, particularly the bank account entity (model). When using the approach I have just described, you will have bank account 'table' that looks like this:

| Id  | AccountNumber | Balance | DateCreated | LastUpdated |
| --- | ------------- | ------- | ----------- | ----------- |
| 1   | 12345         | 1000    | 01-01-2020  | 31-01-2024  |

If you are asked to explain what's going on with bank account 1, here is what you will probably come up with:

- The account was created on on 01-01-2020.
- The current balance is $1000.00.
- It was last updated (the balance maybe?) on 31-01-2024.

There is not much information or insight that you can get from this table. For instance, how did the current balance come to the $1000.00 we are looking at? You could possibly maintain a separate 'audit' table that records all the changes that have happened but there is another approach, possibly a better approach, depending on who you ask and the use case of course. That approach is event sourcing. By the end of this read, you will have an understanding of what event sourcing is, it's advantages and drawbacks, as well as the use cases where it's applicable.

## What Is Event Sourcing?

Event sourcing is a software design pattern that focuses on recording and storing an application's state as a series of _immutable_ events. Rather than storing the current state of an entity (model), event sourcing maintains a log of events that led to the entity's current state. These events represent meaningful occurrences within the system, such as user actions, system events, or business processes. If you want to get the current state of an entity, you just simply replay all of it's associated events. Event sourcing allows us to not only see the current state in our application, but it also shows us how we got to the current state.

> It is more meaningful to record the user's actions as immutable events, rather than recording the effect of those actions on a mutable database…
>
> -- _M. Kleppmann: Designing Data-Intensive Applications, 1st edition_

In our bank account example, you can have events like `AccountCreated`, `AmountDeposited`, `AmountWithdrawn`, etcetera. Whenever an event occurs, it is stored in an _append-only_ 'database' (event store), from which they will be replayed when needed. I need to emphasise of the _append-only_ part. Once persisted, an event cannot be deleted or updated.

Before I talk about the advantages of this approach, I want to explain a few characteristics of events that make them quite useful.

### A Look At Events

At the core of a system that uses event sourcing are the events themselves. There are a few characteristics that events have, which make them a compelling alternative to state management.

- **Events are facts**: Events describe what happened in your system as a matter of fact. For instance, `AccountCreated` means that some business process occurred and an account was created.
- **Events are immutable**: Once an event has been created, it cannot be mutated, otherwise that's akin to rewriting history. If an event was wrong, a compensating event will have to be created. You should never update or delete an event. It's a fact that will never change.
- **We describe actions in our systems as series of events**: Whether you're using event sourcing or not, I can argue that we all talk about things that happen in our systems as events. That's the language everyone, technical and non technical, understands. For instance, you can go to anyone in our fictitious bank and say _A bank account was **created**_, or _That loan application was **approved**_, and they will understand exactly what you mean. Events are the language we use in our everyday interactions.

These characteristics are what makes event sourcing a viable approach to managing application state. Let's now take a look at some of the advantages of event sourcing.

## Advantages of Event Sourcing

- **Full history and traceability**: By storing events, developers retain a complete history of how application state has evolved over time. This historical trail offers unparalleled traceability, facilitating auditing, debugging, and compliance requirements.

- **Events typically have meaning for a domain expert**: Like I said earlier, events are the language we use when talking about what's going on in our systems. This is also true for domain experts.
- **Ability to materialise different views**: Since the application's state is stored as a series of events, we can replay them and 'create' any view of our data we want. For instance, you may have have a new requirement to display the the total deposits and withdrawals for a given month. When using the 'traditional' approach of just storing the state and updating it in-place, you will then need to create another table where you will start to keep track of the transactions. With event sourcing, all you need is to replay the deposit and withdrawal events to get the values you need.
- **Easy reporting**: With event sourcing, generating reports becomes a breeze. This is enabled by the ability to materialise any view of the data from the events. These materialised views or projections can be generated just-in-time or can be updated as the events occur.
- **Makes it easier to evolve applications**: Event sourcing decouples the storage of the application state from its representation. This significantly makes it easy to evolve how your application state is represented.
- **Event-driven architectures**: We can use the events generated in our system as messages to communicate with external systems or components, and trigger actions based on those events. This can improve the scalability and performance of our system.

It would be very naive to think that event sourcing does not have any drawbacks, or at least tradeoffs one needs to consider. Here are some of the challenges or drawbacks of event sourcing:

- **Event schema evolution**: As the application evolves, the schema of our events may change. When this happens, one needs to ensure that there are no compatibility issues between old and new event schemas.
- **Eventual consistency**: Since the application state is derived from a series of events, it may take time to process and apply the events.
- **Event store management**: The event store is an append-only store for all the events that occur in our system. This means it will grow indefinitely so it needs to be scalable and reliable. In addition, as the number of events grow, some queries may experience slight degradation as they have to fetch quite a lot of events to replay. This however is not a big issue in my opinion, as long as proper indexes are set up. In addition, there is a concept of snapshots that solve this problem. We will look at snapshots in a subsequent post.

## Use Cases for Event Sourcing

Here are some of the scenarios where event sourcing can be used:

**Financial Systems**: Event sourcing finds extensive use in financial systems where accurate auditing and compliance are paramount. By logging every financial transaction as an immutable event, organisations can ensure transparent record-keeping, meeting regulatory requirements with ease.

**Logistics management**: Event sourcing fits when in logistics management. One can store all the events that occurred to a particular shipment allowing users of the system to easily track and monitor its status

**IoT and Telemetry Data**: In IoT (Internet of Things) applications, event sourcing proves invaluable for capturing and analyzing telemetry data from sensors and devices. The event log serves as a reliable source of truth for understanding device behavior, detecting anomalies, and deriving actionable insights.

**CQRS (Command Query Responsibility Segregation)**: Event sourcing often complements the CQRS pattern, separating command-side operations (write) from query-side operations (read). The queries will be simply materialising views from the events.

There are numerous examples of where event sourcing can be used as a means to manage application state. Some examples that I didn't expand on include online gaming, healthcare systems and e-commerce.

## Conclusion

Event sourcing is a powerful and flexible pattern for managing state in our applications. It provides a lot of benefits, but it also requires careful design and implementation to overcome some of its drawbacks. Like everything in software engineering, one needs to weigh the pros and cons before picking event sourcing as a means to manage their application state. In the next post, we will talk about how you can implement event sourcing in .NET applications.

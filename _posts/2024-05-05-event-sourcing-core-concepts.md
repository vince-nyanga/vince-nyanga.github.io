---
title: "Event Sourcing: Core Concepts"
excerpt: "In this post, I will explain the core concepts of event sourcing as well as its relationship with Domain-Driven Design (DDD) and Command Query Responsibility Segregation (CQRS)."
date: 2024-05-05
tags: [System Design, Event Sourcing]
---

In the [previous post]({{ site.baseurl}}/event-sourcing), I introduced event sourcing, a software design pattern that focuses on recording and storing an application's state as a series of _immutable_ events. In this post, I will explain the core concepts of event sourcing. These concepts are essential to understanding how event sourcing works and how it can be applied in your applications. In addition, I will discuss how event sourcing relates to Domain-Driven Design (DDD) and Command Query Responsibility Segregation (CQRS), two other design patterns that are often used in conjunction with event sourcing.

## Core Concepts

Here are the core concepts of event sourcing:

### Event

As explained in the previous post, an event is a _fact_ that describes something that happened in your system. Events are _immutable_, meaning that once they are created, they cannot be changed. Events represent meaningful occurrences within the system, such as user actions, system events, or business processes. Examples of events include `AccountCreated`, `AmountDeposited`, `AmountWithdrawn`, etc.

### Event Stream

An event stream is a sequence of events that are related to a specific entity or aggregate. Events in an event stream are ordered by their occurrence, with the first event being the oldest and the last event being the most recent. Event streams are _append-only_, meaning that events can only be added to the stream and never removed or modified.

In our previous example, each bank account would have its own event stream that contains events related to that account, such as `AccountCreated`, `AmountDeposited`, `AmountWithdrawn`, etc.

### Event Store

An event store is a database that stores events in an append-only fashion. Event stores are designed to efficiently store and retrieve events, allowing applications to replay events to reconstruct the state of an entity or aggregate. Event stores are typically implemented as a log-structured storage system, optimized for write-heavy workloads.

### Projection

A projection is a denormalized view of the data stored in the event store. Projections are used to materialize different views of the data, allowing applications to query and display data in a format that is optimized for read operations. Projections are updated by replaying events from the event store and applying them to the projection.

Projections have the advantage of being optimized for read operations, as they can be precomputed and stored in a format that is easy to query. This allows applications to generate reports, display data on user interfaces, or perform analytics without having to query the event store directly.

### Snapshot

A snapshot is a point-in-time representation of the state of an entity or aggregate. Snapshots are used to optimize the replay of events by reducing the number of events that need to be applied to reconstruct the state. Snapshots are taken periodically or when a certain threshold of events is reached, allowing applications to start the replay from the snapshot instead of the beginning of the event stream.

## Event Sourcing Workflow

The typical workflow of an event sourcing application involves the following steps:

1. **Handle Command**: When a user interacts with the application, a command is generated that represents the user's intent. The command is sent to the command handler, which validates the command and produces one or more events.
2. **Persist Events**: The events produced by the command handler are persisted to the event store in an append-only fashion. The events are stored in an event stream that represents the entity or aggregate being modified.
3. **Update Projections**: The events are replayed from the event store and applied to the projections to update the read models. The projections are optimized for read operations and provide a denormalized view of the data. Projection updates can be done synchronously or asynchronously, depending on the requirements of the application.

## How State Is Derived From Events

The state of an entity or aggregate in an event sourcing application is derived by replaying the events that are related to that entity. The state is reconstructed by applying the events in the order they occurred, starting from an initial state (usually empty) and applying each event to update the state. It is basically a fold operation over the event stream. Here is an oversimplified example in C#:

```csharp
var state = events.Aggregate(initialState, (state, @event) => state.Apply(@event));
```

The `Apply` method is a function that takes an event and updates the state of the entity or aggregate based on the event. By replaying the events in the event stream, the application can reconstruct the state of the entity at any point in time.

For mathematical folks, you can think of the state of an entity at a given time as the integral of the event stream up to that time:

$$state(now) = \int_{t=0}^{now} stream(t) dt$$

## Event Sourcing And Domain-Driven Design

Event sourcing is closely related to the principles of Domain-Driven Design (DDD), a design approach that focuses on modeling the domain of a software system in a way that reflects the business requirements and constraints. Event sourcing aligns well with DDD by providing a mechanism for capturing and storing domain events, which can be used to reconstruct the state of the domain objects.

This does not mean that you have to use DDD to use event sourcing. Event sourcing can be used in any application that can benefit from capturing and storing events to represent the state of the system.

## Event Sourcing And CQRS

CQRS is a design pattern that separates the read and write operations of an application into distinct components. In event sourcing, CQRS is often used to separate the command side (handling commands and producing events) from the query side (querying projections and displaying data). This separation allows applications to optimize for read and write operations independently, improving performance and scalability.

Like with DDD, event sourcing can be used with or without CQRS. However, the two patterns complement each other well and are often used together to build complex, scalable, and maintainable systems.

### Conclusion

In this post, I have explained the core concepts of event sourcing, including events, event streams, event stores, projections, and the event sourcing workflow. I have also discussed how event sourcing relates to Domain-Driven Design (DDD) and Command Query Responsibility Segregation (CQRS). Understanding these concepts is essential for implementing event sourcing in your applications and leveraging its benefits for building scalable and maintainable systems. In the next post, I will show a practical example of how event sourcing can be implemented in a simple application. Stay tuned!

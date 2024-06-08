---
title: "Event Sourcing: Snapshots"
excerpt: "In this post, I will talk about snapshots in event sourcing and how they can be used to optimize read performance in an event-sourced system."
date: 2024-06-08
tags: [System Design, Event Sourcing]
---

In the [previous post]({{ site.baseurl}}/event-sourcing-sample), I built a simple bank account application using event sourcing. The application showcased the basic principles of event sourcing. In this post, I will build on the previous code to add snapshots. First, let's understand what snapshots are and why they can be valuable in an event-sourced system.

## What Are Snapshots?

In event sourcing, snapshots are a way to optimize the read performance of an event-sourced system. Instead of replaying all events from the beginning of time to reconstruct the current state of an entity, snapshots capture the state of the entity at a specific point in time. When the entity needs to be rehydrated, the latest snapshot is loaded along with the events that occurred after the snapshot was taken. This reduces the number of events that need to be replayed, improving the performance of read operations.

A snapshot consists of the state of the entity and the version of the entity at the time the snapshot was taken. The version is used to determine which events need to be applied to bring the entity up to date.

## When to Take Snapshots?

The decision of when to take snapshots depends on the size of the event log and the performance requirements of the system. If the event log grows too large, replaying all events to reconstruct the entity's state can become slow. In this case, taking snapshots periodically can help improve read performance. The frequency of taking snapshots can be based on the number of events that have occurred, for instance, after every 20 events. However, taking snapshots too frequently can introduce overhead, so it's essential to strike a balance and find the right snapshotting strategy for your system.

## Implementing Snapshots

Before we implement snapshots, we need to make a few changes to our previous implementation. First, the `BankAccountEvent` record needs to have a `SequenceNumber` property to keep track of the sequence of events.

```csharp
public abstract record BankAccountEvent
{
    public required Guid AccountId { get; init; }
    public required DateTimeOffset Timestamp { get; init; }
    // New property
    public required uint SequenceNumber { get; init; }
}
```

In addition, instead of sorting events by timestamp, we will sort them by sequence number when rehydrating the entity. When loading snapshots, we will also load the sequence number of the snapshot. All the code changes can be found in the [GitHub repository](https://github.com/vince-nyanga/event-sourcing-sample).

Now that we have made the necessary changes, we can implement the snapshotting logic. We will define a class `BankAccountSnapshot` that will hold the state of the bank account and the sequence number of the last event applied to the snapshot.

```csharp
public record BankAccountSnapshot(Guid Id, decimal Balance, Guid OwnerId, AccountStatus Status, uint SequenceNumber);
```

In addition, we will add logic to create a `BankAccount` instance from a snapshot:

```csharp
public record BankAccount
{
    // code omitted for brevity...
    public static BankAccount FromSnapshot(BankAccountSnapshot snapshot) =>
        new()
        {
            Id = snapshot.Id,
            Balance = snapshot.Balance,
            OwnerId = snapshot.OwnerId,
            Status = snapshot.Status
        };
}
```

Let's now turn our attention to the `BankAccountStore`. We will introduce a new field to store the snapshots and a method to save and load snapshots.

```csharp
public class BankAccountStore
{
    private readonly Dictionary<Guid, SortedList<uint, BankAccountEvent>> _eventStreams = new();
    private readonly Dictionary<Guid, SortedList<uint, BankAccountSnapshot>> _snapshots = new();

    public void Append(BankAccountEvent @event)
    {
        if (!_eventStreams.ContainsKey(@event.AccountId))
        {
            _eventStreams.Add(@event.AccountId, new());
        }

        _eventStreams[@event.AccountId].Add(@event.SequenceNumber, @event);

        // if every 5 events, take a snapshot
        if (_eventStreams[@event.AccountId].Count % 5 == 0)
        {
            SaveSnapshot(@event);
        }
    }

    public (BankAccount? BankAccount, uint HighestSequenceNumber) Load(Guid accountId)
    {
        if (!_eventStreams.TryGetValue(accountId, out var stream))
        {
            return (null, 0);
        }

        var snapshot = LoadSnapshot(accountId);
        var bankAccount = snapshot != null ? BankAccount.FromSnapshot(snapshot) : new BankAccount();

        var latestEvents = stream.Values
            .Where(x => x.SequenceNumber > (snapshot?.SequenceNumber ?? 0)).ToArray();

        // apply events to bank account
        bankAccount = latestEvents.Aggregate(bankAccount, (current, @event) => current.When(@event));

        var highestSequenceNumber = latestEvents.Length != 0
            ? latestEvents.Max(x => x.SequenceNumber)
            : snapshot?.SequenceNumber ?? 0;

        return (bankAccount, highestSequenceNumber);
    }

    private BankAccountSnapshot? LoadSnapshot(Guid accountId)
    {
        return !_snapshots.TryGetValue(accountId, out var snapshots)
            ? null : snapshots.Last().Value;
    }

    private void SaveSnapshot(BankAccountEvent @event)
    {
        var (bankAccount, sequenceNumber) = Load(@event.AccountId);
        if (!_snapshots.TryGetValue(@event.AccountId, out var bankAccountSnapshots))
        {
            bankAccountSnapshots = new();
            _snapshots.Add(@event.AccountId, bankAccountSnapshots);
        }

        var snapshot = new BankAccountSnapshot(bankAccount!.Id, bankAccount.Balance, bankAccount.OwnerId, bankAccount.Status, sequenceNumber);
        bankAccountSnapshots.Add(@event.SequenceNumber, snapshot);
    }
}
```

In the `Append` method, we take a snapshot every 5 events by calling the `SaveSnapshot` method. The `Load` method now loads the latest snapshot and applies events that occurred after the snapshot was taken. With these changes, we have successfully introduced snapshots to our event-sourced bank account application. The `CommandHandler` class was also updated to include the `SequenceNumber` property in the events. However, I have omitted the changes here for brevity. You can find the complete code in the [GitHub repository](https://github.com/vince-nyanga/event-sourcing-sample).

## Conclusion

Snapshots are a valuable optimization technique in event sourcing that can improve the read performance of an event-sourced system. By periodically capturing the state of an entity and the sequence number of the last event applied, snapshots reduce the number of events that need to be replayed during rehydration. This can lead to significant performance improvements, especially in systems with large event logs.

When implementing snapshots, it's essential to strike a balance between the frequency of taking snapshots and the overhead they introduce. In the next post, I will talk about projections and how they can be used to materialize different views of the data stored in the event store. Stay tuned!

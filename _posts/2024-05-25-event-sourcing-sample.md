---
title: "Event Sourcing: Example"
excerpt: "In this post, I will show a basic example of event sourcing."
date: 2024-05-25
tags: [System Design, Event Sourcing]
---

In the [previous post]({{ site.baseurl}}/event-sourcing-core-concepts), I introduced the core concepts of event sourcing, including the relationship between event sourcing and domain-driven design and CQRS. In this post, I will show a basic example of event sourcing. We will implement a simple bank account application using event sourcing. This will be a basic console application which we will build upon in future posts.

Let's first define the events that will occur in our bank account application.

## Events

1. `AccountOpened`: This event represents the creation of a new bank account.
2. `AccountApproved`: This event represents the approval of a bank account, which allows transactions to be made.
3. `FundsDeposited`: This event represents the deposit of funds into a bank account.
4. `FundsWithdrawn`: This event represents the withdrawal of funds from a bank account.

Here are the events defined in C#:

```csharp
public abstract record BankAccountEvent
{
    public required Guid AccountId { get; init; }
    public required DateTimeOffset Timestamp { get; init; }
}

public sealed record AccountOpened : BankAccountEvent
{
    public required Guid OwnerId { get; init; }
}

public sealed record AccountApproved : BankAccountEvent
{
    public required Guid ApprovedBy { get; init; }
}

public sealed record FundsDeposited : BankAccountEvent
{
    public required decimal Amount { get; init; }
}

public sealed record FundsWithdrawn : BankAccountEvent
{
    public required decimal Amount { get; init; }
}
```

All events inherit from the `BankAccountEvent` record, which contains common properties such as `AccountId` and `Timestamp`. Each event represents a specific occurrence in the bank account's lifecycle. You will see later on why we have used an abstract record for the base event.

## Commands

Next, we will define the commands that can be issued by users. Commands represent the intent of the user to perform an action on the bank account. Commands are handled by the application and produce events that are stored in the event store.

1. `OpenAccount`: This command is used to open a new bank account.
2. `ApproveAccount`: This command is used to approve a bank account.
3. `DepositFunds`: This command is used to deposit funds into a bank account.
4. `WithdrawFunds`: This command is used to withdraw funds from a bank account.

```csharp
public sealed record OpenAccount(Guid OwnerId);

public sealed record ApproveAccount(Guid AccountId, Guid ApprovedBy);

public sealed record DepositFunds(Guid AccountId, decimal Amount);

public sealed record WithdrawFunds(Guid AccountId, decimal Amount);
```

## Bank Account Entity

The bank account entity will be responsible for maintaining the state of the bank account and applying the events to update its state.

```csharp
public record BankAccount
{
    public Guid Id { get; private set; }
    public decimal Balance { get; private set; }
    public Guid OwnerId { get; private set; }
    public AccountStatus Status { get; private set; }

    public BankAccount When(BankAccountEvent @event)
    {
        return @event switch
        {
            AccountOpened accountOpened => Apply(accountOpened),
            AccountApproved accountApproved => Apply(accountApproved),
            FundsDeposited moneyDeposited => Apply(moneyDeposited),
            FundsWithdrawn moneyWithdrawn => Apply(moneyWithdrawn),
            _ => this
        };
    }

    private BankAccount Apply(AccountOpened accountOpened) =>
        this with { Id = accountOpened.AccountId, OwnerId = accountOpened.OwnerId, Balance = 0, Status = AccountStatus.Pending };

    private BankAccount Apply(AccountApproved accountApproved) =>
        this with { Status = AccountStatus.Active };

    private BankAccount Apply(FundsDeposited fundsDeposited) =>
        this with { Balance = Balance + fundsDeposited.Amount };

    private BankAccount Apply(FundsWithdrawn fundsDeposited) =>
        this with { Balance = Balance - fundsDeposited.Amount };
}

public enum AccountStatus
{
    Pending,
    Active
}
```

The `BankAccount` record represents the state of a bank account. The `When` method is used to apply events to the bank account entity and update its state. The `Apply` methods are used to apply specific events to the bank account entity and return a new instance with the updated state.

## Event Store

The event store will be responsible for storing and retrieving events related to bank accounts. For this example, we will use an in-memory event store.

```csharp
public class BankAccountStore
{
    private readonly Dictionary<Guid, SortedList<DateTimeOffset, BankAccountEvent>> _eventStreams = new();

    public void Append(BankAccountEvent @event)
    {
        if (!_eventStreams.ContainsKey(@event.AccountId))
        {
            _eventStreams.Add(@event.AccountId, new());
        }

        _eventStreams[@event.AccountId].Add(@event.Timestamp, @event);
    }

    public BankAccount? Load(Guid accountId)
    {
        if (!_eventStreams.TryGetValue(accountId, out var stream))
        {
            return null;
        }

        var events = stream.Values;

        return events.Aggregate(new BankAccount(), (current, @event) => current.When(@event));
    }
}
```

Our in-memory event store uses a dictionary to store event streams for each bank account. The event streams are sorted by their timestamp to ensure that events are applied in the correct order when reconstructing the bank account state. The `Append` method is used to add events to the event store, while the `Load` method is used to load the bank account state by replaying events from the event stream.

Like I explained in the previous post, the state of the bank account is derived by replaying the events in the event stream and applying them to the bank account entity. This is a simple left fold operation over the event stream.

## Command Handler

The command handler is responsible for processing commands and producing events that are stored in the event store. The command handler validates the commands and produces events based on the command type. Here is the implementation of the command handler:

```csharp
public class CommandHandler(BankAccountStore store)
{
    public Guid Handle(OpenAccount command)
    {
        var accountOpened = new AccountOpened
        {
            AccountId = Guid.NewGuid(),
            OwnerId = command.OwnerId,
            Timestamp = DateTimeOffset.UtcNow
        };

        store.Append(accountOpened);
        return accountOpened.AccountId;
    }

    public void Handle(ApproveAccount command)
    {
        var account = GetAccount(command.AccountId);
        if (account.Status != AccountStatus.Pending)
        {
            throw new Exception("Account not pending approval");
        }

        var accountApproved = new AccountApproved
        {
            AccountId = command.AccountId,
            ApprovedBy = command.ApprovedBy,
            Timestamp = DateTimeOffset.UtcNow
        };

        store.Append(accountApproved);
    }

    public void Handle(DepositFunds command)
    {
        var account = GetAccount(command.AccountId);

        EnsureAccountIsActive(account);

        var fundsDeposited = new FundsDeposited
        {
            AccountId = command.AccountId,
            Amount = command.Amount,
            Timestamp = DateTimeOffset.UtcNow
        };

        store.Append(fundsDeposited);
    }

    public void Handle(WithdrawFunds command)
    {
        var account = GetAccount(command.AccountId);

        EnsureAccountIsActive(account);

        if (account.Balance < command.Amount)
        {
            throw new Exception("Insufficient funds");
        }

        var fundsWithdrawn = new FundsWithdrawn
        {
            AccountId = command.AccountId,
            Amount = command.Amount,
            Timestamp = DateTimeOffset.UtcNow
        };

        store.Append(fundsWithdrawn);
    }

    private static void EnsureAccountIsActive(BankAccount account)
    {
        if (account.Status != AccountStatus.Active)
        {
            throw new Exception("Account not active");
        }
    }

    private BankAccount GetAccount(Guid accountId)
    {
        var account = store.Load(accountId);
        if (account is null)
        {
            throw new Exception("Account not found");
        }

        return account;
    }
}
```

## Putting It All Together

Now that we have defined the events, commands, bank account entity, event store, and command handler, we can put it all together in a simple console application. Here is an example of how you can use the event sourcing components to interact with bank accounts:

```csharp
var bankAccountStore = new BankAccountStore();
var commandHandler = new CommandHandler(bankAccountStore);

var accountId = commandHandler.Handle(new OpenAccount(Guid.NewGuid()));
commandHandler.Handle(new ApproveAccount(accountId, Guid.NewGuid()));
commandHandler.Handle(new DepositFunds(accountId, 100m));
commandHandler.Handle(new WithdrawFunds(accountId, 50m));
commandHandler.Handle(new DepositFunds(accountId, 100m));

var account = bankAccountStore.Load(accountId);

Console.WriteLine(account);
Console.ReadLine();
```

This is a basic example of how you can implement event sourcing in a simple bank account application. In subsequent posts, we will explore more advanced topics and features, such as snapshots, and projections. Stay tuned!

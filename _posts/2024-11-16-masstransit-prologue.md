---
title: "Building A Message-Based System Using MassTransit - Prologue"
excerpt: "In this post, I will introduce the project we will be building in the MassTransit series."
date: 2024-11-16
tags: [MassTransit, Messaging, Distributed Systems, Design]
---

With the popularity of microservices architecture and distributed systems, the need for efficient communication between services has become paramount. Messaging systems play a crucial role in enabling asynchronous communication, decoupling services, and ensuring fault tolerance in distributed environments. In this blog series, we will explore how to build a message-based system using [MassTransit](https://masstransit-project.com/), a powerful open-source library for building distributed applications in .NET. Here is an overview of what we will cover in this series:

- **Part 1**: Prologue: Introducing HonesRemit (this post)
- **Part 2**: Introduction to MassTransit _(coming soon)_
- **Part 3**: Adding MassTransit to HonesRemit _(coming soon)_
- **Part 4**: Understanding the MassTransit message pipeline _(coming soon)_
- **Part 5**: Implementing outbox pattern with MassTransit _(coming soon)_
- **Part 6**: Implementing saga pattern with MassTransit _(coming soon)_
- **Part 7**: Writing tests for MassTransit _(coming soon)_
- **Part 8**: MassTransit vs NServiceBus _(coming soon)_

All the code for this series can be found on [GitHub](https://github.com/vince-nyanga/hones.remit.api). There will be a few branches that you can check out to see the progress. The `main` branch will always have the latest code.

In this post, I will introduce the project we will be building in this series. At the end of this post, we will have a simple (_non-MassTransit_) web API that we will refactor to use MassTransit in the subsequent posts. Let's get started!

## Introducing HonesRemit

HonesRemit is a fictitious money transfer service that allows users to send money to their loved ones across the globe. The service provides a simple RESTful API that enables users to create transfer orders, pay for them, and let recipients collect the money sent. The system could be be composed of multiple microservices that handle different aspects of the end-to-end workflow, such as order creation, payment processing, and notification services. However, for the sake of simplicity, we will put everything in a single monolith (monoliths are not as bad as they make you believe :smile: ). Let's look at the requirements for HonesRemit.

### Requirements

1. **Create Transfer Order**:

   - Users should be able to create a new transfer order by providing the sender's details, recipient's details, and the amount to be transferred.
   - The system should generate a unique order ID for each transfer order.
   - An email notification should be sent to the sender confirming the order creation.
   - The sender has a limited time to pay for the order, after which it will be canceled.

2. **Pay for Transfer Order**:

   - Users should be able to pay for a transfer order using a payment gateway (out of scope).
   - Some regulatory checks should be performed before the money is ready for collection.
   - An email notification should be sent to both the sender and the recipient once the money is ready for collection.

3. **Collect Money**:
   - Once the recipient receives the notification, they should be able to collect the money from a designated collection point.
   - An email notification should be sent to both the sender and the recipient once the money has been collected.

This obviously is a very simplified version of a money transfer service, but it will serve as a good example for our series. In the subsequent posts, we will start building this system using MassTransit to handle the communication between different parts of the system.

## Initial Implementation

To kick things off, let's create a simple web API that will serve as the foundation for our HonesRemit service. We will use ASP.NET Core to build the API and Entity Framework Core to interact with a PostgreSQL database. The API will expose endpoints for creating transfer orders, paying for orders, and collecting money. We will also include basic email notifications using [smtp4dev](https://github.com/rnwood/smtp4dev) for testing purposes.

### Domain Model

Our domain model consists of one entity, `Order`, which represents a transfer order. Here is how it looks like:

```csharp
public class Order
{
    private static readonly string[] SupportedCurrencies = ["ZAR", "USD", "ZIG"];

    private Order()
    {
    }

    private Order(
        Guid publicId,
        string senderEmail,
        string senderName,
        string recipientEmail,
        string recipientName,
        string currency,
        decimal amount)
    {
        PublicId = publicId;
        SenderEmail = senderEmail;
        SenderName = senderName;
        RecipientEmail = recipientEmail;
        RecipientName = recipientName;
        Currency = currency;
        Amount = amount;
        Status = OrderStatus.Created;
        DateCreatedUtc = DateTimeOffset.UtcNow;
    }

    public long Id { get; init; }
    public Guid PublicId { get; init; }
    public OrderStatus Status { get; private set; }
    public DateTimeOffset DateCreatedUtc { get; init; }
    public DateTimeOffset? DateExpiredUtc { get; private set; }
    public DateTimeOffset? DatePaidUtc { get; private set; }
    public DateTimeOffset? DateCancelledUtc { get; private set; }
    public DateTimeOffset? DateCollectedUtc { get; private set; }
    public string SenderEmail { get; init; } = null!;
    public string SenderName { get; init; } = null!;
    public string RecipientEmail { get; init; } = null!;
    public string RecipientName { get; init; } = null!;
    public string Currency { get; init; } = null!;
    public decimal Amount { get; init; }

    public ErrorOr<Updated> Pay()
    {
        if (Status != OrderStatus.Created)
        {
            return OrderErrors.InvalidStatus;
        }

        Status = OrderStatus.Paid;
        DatePaidUtc = DateTimeOffset.UtcNow;

        return Result.Updated;
    }

    public ErrorOr<Updated> Expire()
    {
        if (Status != OrderStatus.Created)
        {
            return OrderErrors.InvalidStatus;
        }

        Status = OrderStatus.Expired;
        DateExpiredUtc = DateTimeOffset.UtcNow;

        return Result.Updated;
    }

    public ErrorOr<Updated> Cancel()
    {
        if (Status != OrderStatus.Created)
        {
            return OrderErrors.InvalidStatus;
        }

        Status = OrderStatus.Cancelled;
        DateCancelledUtc = DateTimeOffset.UtcNow;

        return Result.Updated;
    }

    public ErrorOr<Updated> Collect()
    {
        if (Status != OrderStatus.Paid)
        {
            return OrderErrors.InvalidStatus;
        }
        Status = OrderStatus.Collected;
        DateCollectedUtc = DateTimeOffset.UtcNow;

        return Result.Updated;
    }

    public static ErrorOr<Order> Create(
        string senderEmail,
        string senderName,
        string recipientEmail,
        string recipientName,
        string currency,
        decimal amount,
        Guid? publicId = default)
    {
        var validationErrors = new List<Error>();
        if (!SupportedCurrencies.Contains(currency))
        {
           validationErrors.Add(OrderErrors.UnsupportedCurrency);
        }

        if (amount <= 0)
        {
            validationErrors.Add(OrderErrors.AmountMustBePositive);
        }

        if (string.IsNullOrEmpty(senderEmail))
        {
            validationErrors.Add(OrderErrors.SenderEmailRequired);
        }

        if (string.IsNullOrEmpty(senderName))
        {
            validationErrors.Add(OrderErrors.SenderNameRequired);
        }

        if (string.IsNullOrEmpty(recipientEmail))
        {
            validationErrors.Add(OrderErrors.RecipientEmailRequired);
        }

        if (string.IsNullOrEmpty(recipientName))
        {
            validationErrors.Add(OrderErrors.RecipientNameRequired);
        }

        if (validationErrors.Count > 0)
        {
            return validationErrors;
        }

        return new Order(
            publicId ?? Guid.NewGuid(),
            senderEmail,
            senderName,
            recipientEmail,
            recipientName,
            currency,
            amount
        );
    }
}
```

I am not going into detail about the domain model as it is not the focus of this series. Next, we will create the endpoints that we will be using. We are going to use .NET's minimal APIs:

```csharp
public static class OrdersApi
{
    public static void MapOrders(this IEndpointRouteBuilder routes)
    {
        group.MapPost("/", ApiHandler.AddOrder)
            .WithName("AddOrder")
            .Accepts<CreateOrderDto>(contentType: "application/json")
            .Produces<OrderDto>(statusCode: (int)HttpStatusCode.Created)
            .Produces<ProblemDetails>(statusCode: (int)HttpStatusCode.BadRequest)
            .WithOpenApi();

        group.MapPatch("/{orderId:guid}/pay", ApiHandler.PayOrder)
            .WithName("PayOrder")
            .Produces((int)HttpStatusCode.OK)
            .Produces<ProblemDetails>((int)HttpStatusCode.NotFound)
            .Produces<ProblemDetails>((int)HttpStatusCode.BadRequest)
            .WithOpenApi();

        group.MapPatch("/{orderId:guid}/collect", ApiHandler.CollectOrder)
            .WithName("CollectOrder")
            .Produces((int)HttpStatusCode.OK)
            .Produces<ProblemDetails>((int)HttpStatusCode.NotFound)
            .Produces<ProblemDetails>((int)HttpStatusCode.BadRequest)
            .WithOpenApi();

        // other endpoints omitted for brevity
    }

    private sealed class ApiHandler
    {
        public static async Task<IResult> AddOrder(
            OrdersDbContext dbContext,
            IEmailService emailService,
            CreateOrderDto createOrderDto,
            CancellationToken cancellationToken)
        {
            var orderResult = Order.Create(
                createOrderDto.SenderEmail,
                createOrderDto.SenderName,
                createOrderDto.RecipientEmail,
                createOrderDto.RecipientName,
                createOrderDto.Currency,
                createOrderDto.Amount
            );

            if (orderResult.IsError)
            {
                // TODO: return appropriate error response based on error type...
                return Results.BadRequest();
            }

            var order = orderResult.Value;

            await dbContext.Orders.AddAsync(order, cancellationToken);
            await dbContext.SaveChangesAsync(cancellationToken);

            await SendOrderCreatedEmail(emailService, order);

            return Results.CreatedAtRoute("GetOrderById", new { orderId = order.PublicId }, MapToDto(order));
        }

        public static async Task<IResult> PayOrder(
            OrdersDbContext dbContext,
            IEmailService emailService,
            Guid orderId,
            CancellationToken cancellationToken)
        {
            var order = await dbContext.Orders
                .FirstOrDefaultAsync(o => o.PublicId == orderId, cancellationToken);

            if (order is null)
            {
                return Results.NotFound();
            }

            var result = order.Pay();
            await dbContext.SaveChangesAsync(cancellationToken);

            await result.SwitchAsync(
                _ => SendOrderPaidEmails(emailService, order),
                _ => Task.CompletedTask);

            return result.MatchFirst(
                _ => Results.Ok(),
                error => Results.BadRequest(error.Description)
            );
        }

        public static async Task<IResult> CollectOrder(
            OrdersDbContext dbContext,
            IEmailService emailService,
            Guid orderId,
            CancellationToken cancellationToken)
        {
            var order = await dbContext.Orders
                .FirstOrDefaultAsync(o => o.PublicId == orderId, cancellationToken);

            if (order is null)
            {
                return Results.NotFound();
            }

            var result = order.Collect();
            await dbContext.SaveChangesAsync(cancellationToken);

            await result.SwitchAsync(
                _ => SendOrderCollectedEmails(emailService, order),
                _ => Task.CompletedTask);

            return result.MatchFirst(
                _ => Results.Ok(),
                error => Results.BadRequest(error.Description)
            );
        }

        private static async Task SendOrderCreatedEmail(IEmailService emailService, Order order)
        {
            var orderReference = EncodeId(order.Id);
            var emailBuilder = new StringBuilder($"Hi {order.SenderName},")
                .AppendLine()
                .AppendLine()
                .AppendLine("Your order has been created successfully. Please make payment to complete the order.")
                .AppendLine()
                .AppendLine("Order Details:")
                .AppendLine($"Amount: {order.Currency} {order.Amount:N2}")
                .AppendLine($"Reference: {orderReference}")
                .AppendLine($"Recipient: {order.RecipientName} ({order.RecipientEmail})")
                .AppendLine()
                .AppendLine("Thank you for using our service.")
                .AppendLine()
                .AppendLine("Regards,")
                .AppendLine("HonesRemit Team");

            await emailService.SendEmailAsync(order.SenderEmail, $"Order Created - {orderReference}",
                emailBuilder.ToString());
        }

        private static async Task SendOrderPaidEmails(IEmailService emailService, Order order)
        {
            // email sending logic omitted for brevity
            // will send emails to both sender and recipient
        }

        private static async Task SendOrderCollectedEmails(IEmailService emailService, Order order)
        {

        }

        // other methods omitted for brevity
    }
}
```

The above example has 3 endpoints, one for creating an order, one for paying an order, and one for collecting an order. The endpoints are very simple and the code is is pretty straightforward and works well, to a certain extent. This is not the complete implementation, if you want to see the full code, you can check out the starter branch in the [GitHub repository](https://github.com/vince-nyanga/hones.remit.api/tree/starter).

Thats it for this post. In the next post, we will introduce MassTransit, looking at what it is, the good and the bad, and how it can help us build a more robust and scalable system. Stay tuned!

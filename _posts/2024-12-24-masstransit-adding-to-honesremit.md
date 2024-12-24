---
title: "Building A Message-Based System Using MassTransit - Adding MassTransit To HonesRemit"
excerpt: "In this post, I will show how to add MassTransit to a .NET application."
date: 2024-12-24
tags: [MassTransit, Messaging, Microservices]
---

This is the third post in the series `"Building A Message-Based System Using MassTransit"`. Here is an overview of what we will cover in this series:

- [Part 1]({{ site.baseurl}}/masstransit-prologue): Prologue: Introducing HonesRemit
- [Part 2]({{ site.baseurl}}/masstransit-introduction): Introduction to MassTransit
- **Part 3**: Adding MassTransit to HonesRemit _(this post)_
- **Part 4**: Understanding the MassTransit message pipeline _(coming soon)_
- **Part 5**: Implementing outbox pattern with MassTransit _(coming soon)_
- **Part 6**: Implementing saga pattern with MassTransit _(coming soon)_
- **Part 7**: Writing tests for MassTransit _(coming soon)_
- **Part 8**: MassTransit vs NServiceBus _(coming soon)_

In the [previous post]({{ site.baseurl}}/masstransit-introduction), I introduced MassTransit, a powerful open-source framework for building message-based applications in .NET and explain some of its core concepts. In this post, we will take a look at how to add MassTransit to HonesRemit, our fictitious money transfer service. By the end of this post, you will have a good understanding of how to integrate MassTransit into a .NET application. Let's get started!

## Adding MassTransit to HonesRemit

In [Part 1]({{ site.baseurl}}/masstransit-prologue), we introduced HonesRemit, a money transfer service that allows users to send money to their loved ones across the globe. We also showed a basic implementation of the API. In this post, we will add MassTransit to HonesRemit to enable message-based communication between different components of the system.

The first thing we need to do is to add the MassTransit NuGet packages to our project. MassTransit provides a set of NuGet packages that we can use to add messaging capabilities to our application. To add MassTransit to our project, we need to install the following packages:

```bash
dotnet add package MassTransit.RabbitMQ
dotnet add package MassTransit.Extensions.DependencyInjection
```

With these packages installed, we can now start configuring MassTransit in our application. MassTransit provides an extension method `AddMassTransit` that we can use to configure MassTransit in our application. Let's add MassTransit to our application in the `Program.cs` file:

```csharp
builder.Services.AddMassTransit(configurator =>
{
    configurator.SetKebabCaseEndpointNameFormatter();

    var entryAssembly = Assembly.GetEntryAssembly();
    configurator.AddConsumers(entryAssembly);

    configurator.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("localhost", 9520, "/", h =>
        {
            h.Username("guest");
            h.Password("guest");
        });

        cfg.ConfigureEndpoints(context);
    });
});
```

That's it! We have now added MassTransit to our application. Several required interfaces (and their implementations, appropriate for the transport specified) are registered.

We have configured it to use RabbitMq as the messaging transport and added the necessary consumers to the configuration. This means that we will need to have RabbitMq running on our local machine. If you have cloned this repository from [GitHub](https://github.com/vince-nyanga/hones.remit.api/), you will find a `docker-compose.yml` file that you can use to start RabbitMq. Simply run the following command to start RabbitMq:

```bash
docker-compose up -d
```

Now that we have MassTransit configured, we need to start refactoring our code to make use of MassTransit. I will not show all the changes in this post, instead, I will highlight a few MassTransit features that I think are important for you to know. The first feature that I would like to highlight is the Request-Reply pattern.

## Request-Reply Pattern Using MassTransit

The Request-Reply pattern is a common messaging pattern used in distributed systems. A component sends a message (request) to another component and waits for a response. If you have have made an HTTP or GRPC request, then you have used a synchronous form of Request-Reply pattern.

MassTransit provides abstractions for implementing the Request-Reply pattern using queues. In this case, the sender sends a message to a queue. The receiver gets the message, and, once done processing the message, sends a reply to the sender via a queue. The sender then listens on the reply queue for the response.

Why would you choose to implement Request-Reply over a queue? There are many reasons actually, which include the ability to handle failures, scalability, and the ability to decouple components.

In our HonesRemit application, we are going to illustrate the Request-Reply pattern to implement the creation of a new order. We are going to create a `CreateOrder` message:

```csharp
public record CreateOrder
{
    public required string SenderEmail { get; init; }
    public required string SenderName { get; init; }
    public required string RecipientEmail { get; init; }
    public required string RecipientName { get; init; }
    public required string Currency { get; init; }
    public required decimal Amount { get; init; }
}
```

Since this is not going to be a simple fire-and-forget message, we are going have to create response messages that will be returned to the sender by the consumer of the `CreateOrder` message. We are going to create a `NewOrderResult` and `OrderCreationFailedResult` messages:

```csharp
public record NewOrderResult
{
    public required Guid Id { get; init; }
    public required string Reference { get; init; }
    public required string Status { get; init; }
    // other properties omitted for brevity
}

public record OrderCreationFailedResult
{
    public required string Error { get; init; }
}
```

Now we are going to create a consumer for the `CreateOrder` message:

```csharp
public class CreateOrderConsumer : IConsumer<CreateOrder>
{
    private readonly ILogger<CreateOrderConsumer> _logger;
    private readonly OrdersDbContext _dbContext;

    public CreateOrderConsumer(ILogger<CreateOrderConsumer> logger, OrdersDbContext dbContext)
    {
        _logger = logger;
        _dbContext = dbContext;
    }

    public async Task Consume(ConsumeContext<CreateOrder> context)
    {
        _logger.LogInformation("Creating order: {@Order}", context.Message);

        var orderResult = Order.Create(
            context.Message.SenderEmail,
            context.Message.SenderName,
            context.Message.RecipientEmail,
            context.Message.RecipientName,
            context.Message.Currency,
            context.Message.Amount
        );

        if (orderResult.IsError)
        {
            await context.RespondAsync(new OrderCreationFailedResult
            {
                Error = orderResult.FirstError.Description
            });

            return;
        }

        var order = orderResult.Value;

        await _dbContext.Orders.AddAsync(order, context.CancellationToken);
        await _dbContext.SaveChangesAsync(context.CancellationToken);

        _logger.LogInformation("Order created: {@Order}", order);

        await context.RespondAsync(new NewOrderResult
        {
            Id = order.PublicId,
            Reference = order.Id.Encode(),
            Status = order.Status.ToString(),
            DateCreatedUtc = order.DateCreatedUtc,
            SenderEmail = order.SenderEmail,
            SenderName = order.SenderName,
            RecipientEmail = order.RecipientEmail,
            RecipientName = order.RecipientName,
            Currency = order.Currency,
            Amount = order.Amount
        });

        await context.Publish(new OrderCreated(order.PublicId), context.CancellationToken);
    }
}
```

As you can see, we are responding to the sender with a `NewOrderResult` message if the order was successfully created. If the order creation failed, we respond with an `OrderCreationFailedResult` message. The sender of the `CreateOrder` message will have to expect a response message of type `NewOrderResult` or `OrderCreationFailedResult`, and handle the response accordingly.

With everything in place, let up update the create order endpoint to make use of MassTransit:

```csharp
public static async Task<IResult> AddOrder(
    IRequestClient<CreateOrder> requestClient,
    CreateOrderDto createOrderDto,
    CancellationToken cancellationToken)
{
    var response = await requestClient.GetResponse<NewOrderResult, OrderCreationFailedResult>(new CreateOrder
    {
        SenderEmail = createOrderDto.SenderEmail,
        SenderName = createOrderDto.SenderName,
        RecipientEmail = createOrderDto.RecipientEmail,
        RecipientName = createOrderDto.RecipientName,
        Currency = createOrderDto.Currency,
        Amount = createOrderDto.Amount
    }, cancellationToken);

    if (response.Is(out Response<NewOrderResult>? newOrderResult))
    {
        var order = newOrderResult.Message;
        return Results.CreatedAtRoute("GetOrderById", new { orderId = order.Id }, order);
    }

    if (response.Is(out Response<OrderCreationFailedResult>? failedResult))
    {
        return Results.BadRequest(failedResult.Message.Error);
    }

    return Results.BadRequest("An error occurred while creating the order.");
}
```

In order to use the Request-Reply pattern, we make use of the `IRequestClient<TRequest>` interface which is used to send a request message and expect any of the possible response types. In this case, we only expect two types of responses: `NewOrderResult` and `OrderCreationFailedResult`. All we need to do is call `requestClient.GetResponse<NewOrderResult, OrderCreationFailedResult>()` and use `reponse.Is(out Response<TResponse>)` to check the response type and handle it accordingly.

This is how you can use the Request-Reply pattern with MassTransit. Next, I am going to show the most common pattern used with MassTransit -- the Publish-Subscribe pattern and how we can use it in the HonesRemit application.

## Publish-Subscribe Pattern Using MassTransit

The publish-subscribe pattern is used to broadcast messages to multiple consumers, usually, signaling that something has happened. If you remember, in the initial implementation, we were sending an email after creating an order in the same process. With the publish-subscribe pattern, we can separate the two processes. Once an order has been created, an event is published and the consumer(s) of that event will process it in a separate process. In this case, we will need a consumer that sends an email. The opportunities that this pattern provides are endless. These include:

- **Decoupling components**: The publisher and the subscriber are decoupled. The publisher does not need to know who the subscribers are and the subscribers do not necessarily need be running when the event is published. They will still receive the event when they are up and running. In our case, the email service can be down and the order will still be created.
- **Scalability**: You can have multiple consumers for the same event. This is useful when you have a lot of processing to do and you want to distribute the load across multiple consumers.
- **Flexibility**: You can add new consumers without changing the publisher. This is useful when you want to add new features without changing the existing code.
- **Reliability**: If a consumer fails to process the event, the message will be retried a couple of times.
- **Speed**: The publisher does not have to wait for the consumers to process the event. It can continue with other tasks.

In MassTransit, messages can be published using the `IPublishEndpoint` interface which you can inject directly as a scoped service or access from the `IBus` singleton or the `ConsumeContext` inside a Consumer. In our consumer above, you may have noticed that after saving the order in the database, we published an `OrderCreated` event using `ConsumeContext`. We are now going to add a consumer for our event that will be responsible for sending an email notification to the sender:

```csharp
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    private readonly OrdersDbContext _dbContext;
    private readonly ILogger<OrderCreatedConsumer> _logger;
    private readonly IEmailService _emailService;

    public OrderCreatedConsumer(
        OrdersDbContext dbContext,
        ILogger<OrderCreatedConsumer> logger,
        IEmailService emailService)
    {
        _dbContext = dbContext;
        _logger = logger;
        _emailService = emailService;
    }

    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        _logger.LogInformation("Order created: {@Order}", context.Message);

        var order = await _dbContext.Orders
            .FirstOrDefaultAsync(x => x.PublicId == context.Message.OrderId,
                context.CancellationToken);

        if (order is null)
        {
            _logger.LogError("Order not found: {OrderId}", context.Message.OrderId);
            return;
        }

        var orderReference = order.Id.Encode();
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

        await _emailService.SendEmailAsync(order.SenderEmail, $"Order Created - {orderReference}",
            emailBuilder.ToString());

        _logger.LogInformation("Order created email sent: {OrderId}", context.Message.OrderId);
    }
}
```

When publishing an event, we have no guarantee that there will be any consumers for the event. All we are simply doing is broadcasting the message to 'whom it may concern'. If there is no consumer for the event, then the message will be discarded. On the other hand, if there could be multiple consumers for the event, all of which will receive the message. This is the power of the publish-subscribe pattern. Let's take a look at the last pattern for this post -- sending a message to a single consumer.

## Sending A Message To A Single Consumer

There are scenarios when you want to send a message to only one consumer. This is usually the case when sending commands. Unlike events, commands are usually sent to a single consumer which we expect to handle the command.

In MassTransit, we can use the `ISendEndpoint` interface to send a message to a single consumer. In order to achieve this however, we need to get the send endpoint associated with the consumer. We can get it by using the `ISendEndpointProvider` interface. Here is an example of how we can send a command to cancel an order:

```csharp
public static async Task<IResult> PayOrder(
    ISendEndpointProvider sendEndpointProvider,
    Guid orderId,
    CancellationToken cancellationToken)
{
    var endpoint = await sendEndpointProvider.GetSendEndpoint(new Uri("queue:pay-order"));
    await endpoint.Send(new PayOrder(orderId), cancellationToken);
    return Results.Accepted();
}
```

As you can see, we are getting the send endpoint of the consumer by using the `ISendEndpointProvider.GetSendEndpoint()` method. This method returns an `ISendEndpoint` that we can use to send a message to the consumer. In this case, we are sending a `PayOrder` message to the consumer that is listening on the `pay-order` queue. The consumer will then process the message and execute the necessary business logic.

These are some of the common patterns you can use with MassTransit. Each pattern has its own use case. In summary, here is when you would use each pattern:

- **Request-Reply**: Use this pattern when you need to send a message and expect a response from the consumer. It is important to note that in MassTransit's implementation, the sender will have to wait for the response.
- **Publish-Subscribe**: Use this pattern when you need to broadcast a message to multiple consumers. There is no guarantee that there will be any consumers for the message. It is useful for publishing events to whoever is interested.
- **Send To Single Consumer**: Use this pattern when you need to send a message to a single consumer. This is usually the case when sending commands, where you expect the consumer to handle the command.

## Conclusion

In this post, I have shown how to add MassTransit to a .NET application. I have also shown how to use some of the common patterns in MassTransit. I have only shown a few snippets from the HoneyRemit API. You can find the full implementation in the [GitHub repository](https://github.com/vince-nyanga/hones.remit.api/tree/masstransit/initial).

In the next post, I will take a deep dive into the MassTransit message pipeline. Stay tuned!

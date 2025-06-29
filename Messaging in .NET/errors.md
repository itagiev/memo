## Работа с ошибками

### Skipped queues

Custom queue using message driven system to handle messages that are intentionally not processed by the consumer. Generated automatically, when consumer disconnects from endpoint \
Generated queue has name of the current queue + suffix _skipped

### Error queues

Generated automatically, when consumer throw an exception \
Generated queue has name of the current queue + suffix _error

### Configuring error queue names

Create custom error queue name formatter

```C#
public class MyCustomErrorQueueNameFormatter : IErrorQueueNameFormatter
{
    public string FormatErrorQueueName(string queueName)
    {
        return "-awesome_error";
    }
}
```

Use that formatter

```C#
services.AddMassTransit(x =>
{
    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.SendTopology.ErrorQueueNameFormatter = new MyCustomErrorQueueNameFormatter();
    });
});
```

### Faults

Fault is an event type that is published when a consumer throws an exception that is not handled not retried or consumed

Обработка ошибок

```C#
// Создаем консьюмера
public class OrderCreatedFaultConsumer : IConsumer<Fault<OrderCreated>>
{
    public async Task Consume(ConsumeContext<Fault<OrderCreated>> context)
    {
        Console.WriteLine($"This is an OrderCreatedFault. The faulted message {context.Message.Message.OrderId}");
        await Task.CompletedTask;
    }
}
```

```C#
// Регистрируем
builder.Services.AddMassTransit(options =>
{
    options.AddConsumer<OrderCreatedFaultConsumer>();

    options.UsingRabbitMq((context, cfg) =>
    {
        // ...
    });
});
```

Прослушка любых ошибок, независимо от типа события или команды

```C#
// Создаем консьюмера
public class AllFaultsConsumer : IConsumer<Fault>
{
    public Task Consume(ConsumeContext<Fault> context)
    {
        Console.WriteLine("All the faults that i listen to");
        return Task.CompletedTask;
    }
}
```

```C#
// Регистрируем
builder.Services.AddMassTransit(options =>
{
    options.AddConsumer<AllFaultsConsumer>();

    options.UsingRabbitMq((context, cfg) =>
    {
        // ...
    });
});
```

#### Turning off fault events

```C#
public class OrderCreatedConsumerDefinition : ConsumerDefinition<OrderCreatedConsumer>
{
    public OrderCreatedConsumerDefinition()
    {
        EndpointName = "order-created";
        Endpoint(options =>
        {
            options.Name = "order-created";
            options.ConcurrentMessageLimit = 10;
        });
    }

    protected override void ConfigureConsumer(IReceiveEndpointConfigurator endpointConfigurator,
        IConsumerConfigurator<OrderCreatedConsumer> consumerConfigurator,
        IRegistrationContext context)
    {
        endpointConfigurator.PublishFaults = false;
    }
}
```
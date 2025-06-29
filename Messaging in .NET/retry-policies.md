## Повторное выполнение

### Retry-policy

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

        // Конфигурация retry-policy
        // Эта политика будет работать только для текущего консьюмера
        consumerConfigurator.UseMessageRetry(options =>
        {
            options.Immediate(2);
        });
    }
}
```

Регистрация retry-policy на уровне брокера

```C#
builder.Services.AddMassTransit(bus =>
{
    // ...
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseMessageRetry(options =>
        {
            options.Interval(2, TimeSpan.FromSeconds(30));
        });
        // ...
    });
});
```

Регистрация retry-policy на уровне queue

```C#
builder.Services.AddMassTransit(bus =>
{
    // ...
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseMessageRetry(options =>
        {
            options.Interval(2, TimeSpan.FromSeconds(30));
        });
        
        cfg.ReceiveEndpoint("order-created", endpoint =>
        {
            endpoint.UseMessageRetry(options =>
            {
                options.Exponential(3,
                    TimeSpan.FromSeconds(10),
                    TimeSpan.FromSeconds(60),
                    TimeSpan.FromSeconds(60 / 3));
            });

            endpoint.ConfigureConsumer<OrderCreatedConsumer>(ctx);
        });
    });
});
```

### Get retry attempt

```C#
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        var retryCount = context.GetRetryAttempt();
        if (retryCount > 0)
        {
            Console.WriteLine($"Attempt number: {retryCount}");
        }
    }
}
```

### Exception filter

```C#
public class CreateOrderConsumerDefinition : ConsumerDefinition<CreateOrderConsumer>
{
    protected override void ConfigureConsumer(
        IReceiveEndpointConfigurator endpointConfigurator,
        IConsumerConfigurator<CreateOrderConsumer> consumerConfigurator,
        IRegistrationContext context)
    {
        consumerConfigurator.UseMessageRetry(options =>
        {
            options.Immediate(3);
            // Ignore this type of exceptions
            options.Ignore<OrderTotalTooSmallException>();
            // Handle this type of exceptions
            options.Handle<HandleAllException>();
        });
    }
}
```

### Redelivery

Что-то вроде батчина для ретраев \
Сначала пройдет 3 retry, через 20 секунд redelivery и еще 3 retry и т.д. согласно политике redelivery \
P.S. Нужен redelivery plugin

```C#
public class CreateOrderConsumerDefinition : ConsumerDefinition<CreateOrderConsumer>
{
    protected override void ConfigureConsumer(
        IReceiveEndpointConfigurator endpointConfigurator,
        IConsumerConfigurator<CreateOrderConsumer> consumerConfigurator,
        IRegistrationContext context)
    {
        consumerConfigurator.UseMessageRetry(options =>
        {
            options.Immediate(3);
            options.Ignore<OrderTotalTooSmallException>();
            options.Handle<HandleAllException>();
        });

        consumerConfigurator.UseDelayedRedelivery(options =>
        {
            options.Intervals(
                TimeSpan.FromSeconds(20),
                TimeSpan.FromSeconds(40),
                TimeSpan.FromSeconds(60));
        });
    }
}
```

### Replaying a message

Внутри rabbit mq копируем payload и другие параметры из, например не доставленного сообщения, и некоторые параметры,
и создаем новое сообщение в нужной очереди

#### Replaying all messages

Включаем shovel plugin, переходим в очередь из которой нужно переместить сообщения, в раздел move messages,
вводим имя очереди, в которую нужно переместить сообщения.

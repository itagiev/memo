## Основы

### Publish

```C#
private readonly IPublishEndpoint _publishEndpoint;
// Создание события
var notifyOrderCreated = _publishEndpoint.Publish(new OrderCreated()
{
    Id = createdOrder.Id,
    OrderId = createdOrder.OrderId,
    CreatedAt = createdOrder.OrderDate,
    TotalAmount = createdOrder.OrderItems.Sum(x => x.Price * x.Quantity)
});
```

### Consumer

```C#
// Создание консьюмера
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        await Task.Delay(1000);
        Console.WriteLine(context.ReceiveContext.InputAddress);
        Console.WriteLine($"I just consumed message with Order ID {context.Message.OrderId}, that was created at {context.Message.CreatedAt}");
    }
}
```

### Регистрация консьюмера

```C#
builder.Services.AddMassTransit(options =>
{
    // Добавить конкретного консьюмера
    options.AddConsumer<OrderCreatedConsumer>();

    // Добавить всех консьюмеров в домене
    // options.AddConsumers(Assembly.GetExecutingAssembly());
});
```

### Наименование очередей/событий

Можно установить `EntityName` атрибут для события

```C#
[EntityName("order-created-by-entity-name-attr")]
public class OrderCreated
{
    public int Id { get; set; }
    public Guid OrderId { get; set; }
    public DateTime CreatedAt { get; set; }
    public Decimal TotalAmount { get; set; }
}
```

```C#
builder.Services.AddMassTransit(options =>
{
    // Kebab case (e.g. OrderCreated -> order-created)
    options.SetKebabCaseEndpointNameFormatter();

    // Kebab name с префиксом
    options.SetEndpointNameFormatter(new KebabCaseEndpointNameFormatter("my-prefix", false));
});
```

### Конфигурация receive endpoints

Чтобы начать получать сообщения с определенной очереди, если мы знаем ее имя

```C#
options.UsingRabbitMq((context, cfg) =>
{
    // Такая конфигурация на уровне транспорта перекроет определение консьюмера на уровне bus
    // options.AddConsumer<OrderCreatedConsumer>();
    cfg.ReceiveEndpoint("order-created", endpoint =>
    {
        endpoint.ConfigureConsumer<OrderCreatedConsumer>(context);
    });

    // Тут важен порядок, сначала ReceiveEndpoint, потом ConfigureEndpoints
    cfg.ConfigureEndpoints(context);
});
```

When configuring endpoints manually, ConfigureEndpoints should be excluded or be called after any explicitly configured receive endpoints (Документация)


### Consumer definition

Пример

```C#
// Создание консьюмера
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        await Task.Delay(1000);
        Console.WriteLine(context.ReceiveContext.InputAddress);
        Console.WriteLine($"I just consumed a message with order id {context.Message.OrderId}, that was created at {context.Message.CreatedAt}");
    }
}
// Создание определения консьюмера
public class OrderCreatedConsumerDefinition : ConsumerDefinition<OrderCreatedConsumer>
{
    public OrderCreatedConsumerDefinition()
    {
        EndpointName = "order-created-cons-def";
        Endpoint(options =>
        {
            options.Name = "order-created-cons-def";
            options.ConcurrentMessageLimit = 10;
        });
    }

    protected override void ConfigureConsumer(IReceiveEndpointConfigurator endpointConfigurator,
        IConsumerConfigurator<OrderCreatedConsumer> consumerConfigurator,
        IRegistrationContext context)
    {
        consumerConfigurator.UseMessageRetry(options =>
        {
            options.Immediate(5);
        });
    }
}
// Подключение
builder.Services.AddMassTransit(options =>
{
    //options.AddConsumer<OrderCreatedConsumer>();
    options.AddConsumer<OrderCreatedConsumer, OrderCreatedConsumerDefinition>();
});
```

### Headers

Пример отправки сообщения со своим header value

```C#
var notifyOrderCreated = _publishEndpoint.Publish(new OrderCreated
{
    Id = createdOrder.Id,
    OrderId = createdOrder.OrderId,
    CreatedAt = createdOrder.OrderDate,
    TotalAmount = createdOrder.OrderItems.Sum(x => x.Price * x.Quantity)
},
context =>
{
    context.Headers.Set("my-custom-header", "super-header");
});
```

### Message Expiration

Пример использования TimeToLive

```C#
var notifyOrderCreated = _publishEndpoint.Publish(new OrderCreated
{
    Id = createdOrder.Id,
    OrderId = createdOrder.OrderId,
    CreatedAt = createdOrder.OrderDate,
    TotalAmount = createdOrder.OrderItems.Sum(x => x.Price * x.Quantity)
},
context =>
{
    // Это значит, что через 15 секунд rabbitmq удалит сообщение из брокера, если оно не было ни кем обработано
    context.TimeToLive = TimeSpan.FromSeconds(15);
});
```

### Competing Consumers

Если к одной очереди подключено несколько консьюмеров

Queue: order-created
Exchange: order-created-notification (receive endpoint: order-created)
Exchange: order-created-consumer (receive endpoint: order-created)

Тогда, они будут по очереди обрабатывать сообщения из очереди, т.о. можно скейлить приложение

### Sending commands

[Send (MassTransit Documentation)](https://masstransit.io/documentation/concepts/producers#send)

```C#
ISendEndpointProvider sendEndpointProvider;
var sendEndpoint = await _sendEndpointProvider.GetSendEndpoint(new Uri("queue:create-order-command"));
await sendEndpoint.Send(model);
```

Создаем консьюмера

```C#
public class CreateOrderConsumer : IConsumer<OrderModel>
{
    private readonly IMapper _mapper;
    private readonly IOrderService _orderService;

    public CreateOrderConsumer(IMapper mapper, IOrderService orderService)
    {
        _mapper = mapper;
        _orderService = orderService;
    }

    public async Task Consume(ConsumeContext<OrderModel> context)
    {
        Console.WriteLine($"I got a command to create an order: {context.Message}");

        var orderToAdd = _mapper.Map<Order>(context.Message);
        var createdOrder = await _orderService.AddOrderAsync(orderToAdd);

        var notifyOrderCreated = context.Publish(new OrderCreated
        {
            Id = createdOrder.Id,
            OrderId = createdOrder.OrderId,
            CreatedAt = createdOrder.OrderDate,
            TotalAmount = createdOrder.OrderItems.Sum(x => x.Price * x.Quantity)
        }, context =>
        {
            context.Headers.Set("my-custom-header", "super-header");
            context.TimeToLive = TimeSpan.FromSeconds(30);
        });

        await Task.CompletedTask;
    }
}
```

Регистрируем консьюмера

```C#
services.AddMassTransit(x =>
{
    var entryAssembly = Assembly.GetEntryAssembly();

    x.AddConsumers(entryAssembly);

    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.ReceiveEndpoint("create-order-command", endpoint =>
        {
            endpoint.ConfigureConsumer<CreateOrderConsumer>(context);
        });

        cfg.ConfigureEndpoints(context);
    });
});
```

### Request/Response

Создаем нужный нам консьюмер, который будет использоваться как request

```C#
// Результат работы косьюмера
public class OrderResult
{
    public int Id { get; set; }

    public DateTime OrderDate { get; set; }

    public OrderStatus Status { get; set; }
}
// Если заказ не найден консьюмер вернет результат - ошибку
public class OrderNotFoundResult
{
    public required string ErrorMessage { get; set; }
}
// Поля который мы передаем в consumer
public record VerifyOrder
{
    public int Id { get; set; }
}

public class VerifyOrderConsumer : IConsumer<VerifyOrder>
{
    private readonly IOrderService _orderService;

    public VerifyOrderConsumer(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public async Task Consume(ConsumeContext<VerifyOrder> context)
    {
        // Мы можем проверить, ожидает ли вызывающая сторона тип Order
        if (!context.IsResponseAccepted<Order>())
        {
            throw new ArgumentException(nameof(context));
        }

        var order = await _orderService.GetOrderAsync(context.Message.Id);

        if (order is null)
        {
            await context.RespondAsync(new OrderNotFoundResult
            {
                ErrorMessage = $"Order '{context.Message.Id}' not found"
            });
        }
        else
        {
            await context.RespondAsync(new OrderResult
            {
                Id = context.Message.Id,
                OrderDate = order.OrderDate,
                Status = order.Status
            });
        }
    }
}
```

Регистрируем

```C#
builder.Services.AddMassTransit(options =>
{
    options.AddConsumer<VerifyOrderConsumer>();
    options.AddRequestClient<VerifyOrder>();
});
```

Использование

```C#
IRequestClient<VerifyOrder> _verifyOrderRequestClient;

// GET: api/orders/{id}
[HttpGet("{id}")]
public async Task<ActionResult<Order>> GetOrder(int id)
{
    // Клиент ожидает следующие результаты OrderResult, OrderNotFoundResult и Order,
    // если Order не ожидается, consumer сгенерирует исключение
    var response = await _verifyOrderRequestClient.GetResponse<OrderResult, OrderNotFoundResult, Order>(new VerifyOrder
    {
        Id = id
    });

    if (response.Is(out Response<OrderResult>? orderResult))
    {
        return Ok(orderResult.Message);
    }

    if (response.Is(out Response<OrderNotFoundResult>? error))
    {
        return NotFound(error.Message);
    }

    return BadRequest();
}
```

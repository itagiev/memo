## Saga pattern

Sequence of the local transactions that are coordinated using messaging

### Setup

Создаем модель данных `SagaStateMachineInstance`

```C#
public class OrderStateData : SagaStateMachineInstance
{
    public Guid CorrelationId { get; set; }

    public required string CurrentState { get; set; }

    public Guid OrderId { get; set; }

    [Column(TypeName = "decimal(18,2)")]
    public decimal Amount { get; set; }

    public DateTime? CreatedAt { get; set; }

    public DateTime? PaidAt { get; set; }

    public bool IsBilled { get; set; }

    public OrderStatus OrderStatus { get; set; }

    public DateTime? CanceledAt { get; set; }
}

public class OrderStateMap : IEntityTypeConfiguration<OrderStateData>
{
    public void Configure(EntityTypeBuilder<OrderStateData> builder)
    {
        builder.HasKey(x => x.OrderId);
        builder.Property(x => x.CurrentState).HasMaxLength(64);
        builder.Property(x => x.OrderId);
        builder.Property(x => x.Amount);
        builder.Property(x => x.CreatedAt);
        builder.Property(x => x.PaidAt);
        builder.Property(x => x.CanceledAt);
    }
}
```

Допустим есть следующий `DbContext`. Расширяем его частичным методом `OnModelCreatingPartial`

```C#
public partial class OrderContext : DbContext
{
    public OrderContext(DbContextOptions<OrderContext> options) : base(options)
    {
    }

    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.AddOutboxStateEntity();
        modelBuilder.AddOutboxMessageEntity();
        modelBuilder.AddInboxStateEntity();

        OnModelCreatingPartial(modelBuilder);
    }

    partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
}
```

Расширяем `DbContext`

```C#
public partial class OrderContext : DbContext
{
    public DbSet<OrderStateData> OrderStates { get; set; }

    partial void OnModelCreatingPartial(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.ApplyConfiguration(new OrderStateMap());
    }
}
```

### State machine

Создание проекта `OrderSaga` (worker). \
Создаем класс `OrderStateMachine` от `MassTransitStateMachine<OrderStateData>` \
В нем будут определены состояния, события, и переходы между состояниями

```C#
namespace OrderSaga.StateMachine;
public class OrderStateMachine : MassTransitStateMachine<OrderStateData>
{
    public State Created { get; set; }
    public State Pending { get; set; }
    public State Paid { get; set; }
    public State Canceled { get; set; }

    public Event<OrderCreated> OrderCreated { get; set; }
    public Event<OrderPaid> OrderPaid { get; set; }
    public Event<CancelationRequested> CancelationRequested { get; set; }
    public Event<CancelOrder> CancelOrder { get; set; }

    public OrderStateMachine()
    {
        InstanceState(x => x.CurrentState);

        Event(() => OrderCreated, x => x.CorrelateById(context => context.Message.OrderId));
        Event(() => OrderPaid, x => x.CorrelateById(context => context.Message.OrderId));
        Event(() => CancelationRequested, x => x.CorrelateById(context => context.Message.OrderId));
        Event(() => CancelOrder, x => x.CorrelateById(context => context.Message.OrderId));

        Initially(When(OrderCreated)
            .Then(context =>
            {
                context.Saga.OrderId = context.Message.OrderId;
                context.Saga.Amount = context.Message.TotalAmount;
                context.Saga.OrderStatus = Orders.Domain.Entities.OrderStatus.Pending;
                context.Saga.CreatedAt = context.Message.CreatedAt;
                Console.WriteLine($"Order created {context.Message.OrderId}");
            })
            .TransitionTo(Pending)
        );

        During(Pending,
            When(OrderPaid)
                .Then(context =>
                {
                    context.Saga.OrderId = context.Message.OrderId;
                    context.Saga.Amount = context.Message.AmountPaid;
                    context.Saga.PaidAt = DateTime.UtcNow;
                    context.Saga.IsBilled = false;
                    Console.WriteLine($"Paid order {context.Message.OrderId} of {context.Message.AmountPaid} paid");
                })
                .TransitionTo(Paid)
                .Publish(context => new InvoiceNeeded()
                {
                    TotalAmount = context.Saga.Amount,
                    VAT = context.Saga.Amount * 1.19m,
                    OrderId = context.Message.OrderId // Send and invoice
                }),
            When(CancelationRequested)
                .Then(context =>
                {
                    context.Saga.CanceledAt = DateTime.UtcNow;
                    context.Saga.OrderStatus = Orders.Domain.Entities.OrderStatus.CancelationRequested;
                    Console.WriteLine($"Cancelation requested");
                })
                .TransitionTo(Canceled)
                .Publish(context => new OrderCanceled()
                {
                    OrderId = context.Saga.OrderId
                })
        );

        During(Paid,
            When(CancelationRequested)
                .Then(async context =>
                {
                    context.Saga.OrderId = context.Message.OrderId;
                    context.Saga.PaidAt = DateTime.UtcNow;
                    context.Saga.IsBilled = false;
                    Console.WriteLine($"Canceling the order {context.Message.OrderId}");

                    await context.Publish(new OrderCanceled
                    {
                        OrderId = context.Saga.OrderId
                    });
                })
                .TransitionTo(Canceled)
                .Publish(context => new RefundOrder
                {
                    OrderId = context.Saga.OrderId
                })
        );
    }
}
```

Подключение

```C#
services.AddMassTransit(bus =>
{
    bus.SetKebabCaseEndpointNameFormatter();

    bus.SetEntityFrameworkSagaRepositoryProvider(r =>
    {
        r.ExistingDbContext<OrderContext>();
        r.UseSqlServer();
    });

    bus.AddSagaStateMachine<OrderStateMachine, OrderStateData>()
        .EntityFrameworkRepository(r =>
        {
            r.ExistingDbContext<OrderContext>();
            r.ConcurrencyMode = ConcurrencyMode.Optimistic;
        });

    bus.UsingRabbitMq((context, cfg) =>
    {
        cfg.ConfigureEndpoints(context);
    });
});
```

Если будет попытка выполнить какоето событие или команду которая не подходит под state машину, будет ошибка. Такая ошибка попадет в соответствующую очередь для ошибок.

Ошибки можно игнорировать

```C#
During(Canceled,
    Ignore(CancelationRequested),
    Ignore(OrderPaid));
```

### Finalizing

```C#
// Событие
public class OrderCompleted
{
    public Guid OrderId { get; set; }
}
// Состояние
public State Completed { get; set; }
// Событие
public Event<OrderCompleted> OrderCompleted { get; set; }
// Setup
Event(() => OrderCompleted, x => x.CorrelateById(context => context.Message.OrderId));
DuringAny(When(OrderCompleted).Finalize());
SetCompletedWhenFinalized();
// Вызов
public class OrderCanceledConsumer : IConsumer<OrderCanceled>
{
    public async Task Consume(ConsumeContext<OrderCanceled> context)
    {
        Console.WriteLine($"The order with id: {context.Message.OrderId} was canceled. OrderCanceled event.");

        await context.Publish(new OrderCompleted
        {
            OrderId = context.Message.OrderId
        });
    }
}
// Консьюмер для OrderCompleted не нужен
```

### Scheduling

```C#
public class OrderStateData : SagaStateMachineInstance
{
    //...
    public Guid? PaymentTimeoutTokenId { get; set; }
}

public class OrderStateMachine : MassTransitStateMachine<OrderStateData>
{
    public State AwaitingPayment { get; set; }
    public Event PaymentTimeout { get; set; }
    public Schedule<OrderStateData, PaymentTimeoutExpired> PaymentTimeoutSchedule { get; set; }

    public OrderStateMachine()
    {
        Schedule(() => PaymentTimeoutSchedule, x => x.PaymentTimeoutTokenId, s =>
        {
            s.Delay = TimeSpan.FromSeconds(30);
            s.Received = e => e.CorrelateById(context => context.Message.OrderId);
        });

        Initially(When(OrderCreated)
            .Then(context =>
            {
                context.Saga.OrderId = context.Message.OrderId;
                context.Saga.Amount = context.Message.TotalAmount;
                context.Saga.OrderStatus = Orders.Domain.Entities.OrderStatus.Pending;
                context.Saga.CreatedAt = context.Message.CreatedAt;
                Console.WriteLine($"Order created {context.Message.OrderId}");
            })
            .TransitionTo(Pending)
            .Schedule(PaymentTimeoutSchedule, context => new PaymentTimeoutExpired
            {
                OrderId = context.Message.OrderId
            })
        );

        During(Pending,
            //...
            When(PaymentTimeoutSchedule.Received)
                .Then(context =>
                {
                    Console.WriteLine("Payment timeout occurred, moving to awaiting payment");
                })
                .TransitionTo(AwaitingPayment),
            //...
        );
    }
}
```

Подключение

```C#
services.AddMassTransit(bus =>
{
    //...
    bus.UsingRabbitMq((context, cfg) =>
    {
        cfg.UseInMemoryScheduler();
        cfg.ConfigureEndpoints(context);
    });
});
```

### Saga definitions

```C#
public class OrderSagaDefinition : SagaDefinition<OrderStateData>
{
    public OrderSagaDefinition()
    {
        Endpoint(options =>
        {
            options.Name = "order-saga";
        });
    }
}

services.AddMassTransit(bus =>
{
    //...
    bus.AddSagaStateMachine<OrderStateMachine, OrderStateData, OrderSagaDefinition>()
        .EntityFrameworkRepository(r =>
        {
            r.ExistingDbContext<OrderContext>();
            r.ConcurrencyMode = ConcurrencyMode.Optimistic;
        });
    //...
});
```
## Outbox pattern

Reliable message delivery

### Bus outbox

At least once delivery method

Подключение

```C#
public class OrderContext : DbContext
{
    public OrderContext(DbContextOptions<OrderContext> options) : base(options) { }

    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.AddOutboxStateEntity();
        modelBuilder.AddOutboxMessageEntity();
        modelBuilder.AddInboxStateEntity();
    }
}

builder.Services.AddDbContext<OrderContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
    options.EnableSensitiveDataLogging(builder.Environment.IsDevelopment());
});

builder.Services.AddMassTransit(x =>
{
    x.AddEntityFrameworkOutbox<OrderContext>(o =>
    {
        o.UseSqlServer();
        o.UseBusOutbox();
        // Так мы отключим доставку сообщений, они будут висеть в БД
        // Consumer может забрать их (Inbox)
        o.UseBusOutbox(x => x.DisableDeliveryService());
    });
});
```

Использование

```C#
public async Task AcceptOrder(OrderModel model)
{
    var domainObject = mapper.Map<Order>(model);

    var savedOrder = await this.AddOrderAsync(domainObject);

    var orderReceived = _publishEndpoint.Publish(new OrderReceived()
    {
        CreatedAt = savedOrder.OrderDate,
        OrderId = savedOrder.OrderId
    });

    var notifyOrderCreated = _publishEndpoint.Publish(new OrderCreated()
    {
        CreatedAt = savedOrder.OrderDate,
        OrderId = savedOrder.OrderId,
        TotalAmount = domainObject.OrderItems.Sum(x => x.Price * x.Quantity)
    });

    try
    {
        await _orderRepository.SaveChangesAsync();
    }
    catch (DbUpdateException exception) { }
}
```

### Consumer outbox

Exactly once delivery method

```C#
builder.Services.AddMassTransit(x =>
{
    x.AddEntityFrameworkOutbox<OrderContext>(o =>
    {
        o.DuplicateDetectionWindow = TimeSpan.FromSeconds(30);
        //o.QueryDelay = TimeSpan.FromSeconds(5);
        o.UseSqlServer();
        //o.DisableInboxCleanupService();
        o.UseBusOutbox();
        //o.UseBusOutbox(x => x.DisableDeliveryService());
    });

    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.ReceiveEndpoint("order-created", e =>
        {
            // Указвается, что этот endpoint будет использовать outbox pattern
            e.UseEntityFrameworkOutbox<OrderContext>(context);
            e.ConfigureConsumer<OrderCreatedNotification>(context);
        });

        cfg.ConfigureEndpoints(context);
    });
});

// Можно указать, чтобы все endpoint-ы использовали outbox pattern
x.AddConfigureEndpointsCallback((context, name, cfg) =>
{
    cfg.UseEntityFrameworkOutbox<OrderContext>(context);
});
```
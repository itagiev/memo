## Pipes and filters

Что-то вроде middleware для запросов в asp.net \
Так-же как и с middleware, порядок регистрации фильтров имеет значение

### Send filters

Использование send фильтра

```C#
public class Tenant
{
    public string TenantId { get; set; }
}

// Создание фильтра
public class TenantSendFilter<T> : IFilter<SendContext<T>> where T : class
{
    private readonly Tenant _tenant;

    public TenantSendFilter(Tenant tenant)
    {
        _tenant = tenant;
    }

    public void Probe(ProbeContext context)
    {
        throw new NotImplementedException();
    }

    public Task Send(SendContext<T> context, IPipe<SendContext<T>> next)
    {
        if (!string.IsNullOrEmpty(_tenant.TenantId))
        {
            context.Headers.Set("Tenant-From-Send", _tenant.TenantId);
        }

        return next.Send(context);
    }
}

// Регистрация сервисов, если нужно
services.AddScope<Tenant>();

// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseSendFilter(typeof(TenantSendFilter<>), ctx);
        cfg.ConfigureEndpoints(ctx);
    });
});

// Использование в контроллере
public OrdersController(Tenant tenant)
{
    _tenant = tenant;
    _tenant.TenantId = Guid.NewGuid().ToString();
}

[HttpPost]
public async Task<ActionResult<Order>> PostOrder(OrderModel model)
{
    var sendEndpoint = await _sendEndpointProvider.GetSendEndpoint(new Uri("queue:create-order-command"));
    await sendEndpoint.Send(model);
    return Accepted();
}
```

### Publish filters

```C#
// Создание фильтра
public class TenantPublishFilter<T> : IFilter<PublishContext<T>>
    where T : class
{
    private readonly Tenant _tenant;

    public TenantPublishFilter(Tenant tenant)
    {
        _tenant = tenant;
    }

    public void Probe(ProbeContext context)
    {
        throw new NotImplementedException();
    }

    public Task Send(PublishContext<T> context, IPipe<PublishContext<T>> next)
    {
        if (!string.IsNullOrEmpty(_tenant.TenantId))
        {
            context.Headers.Set("Tenant-From-Publish", _tenant.TenantId);
        }

        return next.Send(context);
    }
}

// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UsePublishFilter(typeof(TenantPublishFilter<>), ctx);
        cfg.ConfigureEndpoints(ctx);
    });
});

// Использование в контроллере
public OrdersController(Tenant tenant)
{
    _tenant = tenant;
    _tenant.TenantId = Guid.NewGuid().ToString();
}

[HttpPost]
public async Task<ActionResult<Order>> PostOrder(OrderModel model)
{
    await _publishEndpoint.Publish(model);
    return Accepted();
}
```

### Consume filters

```C#
// Создание фильтра
public class TenantConsumeFilter<T> : IFilter<ConsumeContext<T>>
    where T : class
{
    private readonly Tenant _tenant;

    public TenantConsumeFilter(Tenant tenant)
    {
        _tenant = tenant;
    }

    public void Probe(ProbeContext context)
    {
        /*
         * Данный метод помогает собирать диагностическую информацию, с помощью него
         * можно задать различные свойства для текущего фильтра
         */
        context.CreateFilterScope("TenantConsume");
    }

    public Task Send(ConsumeContext<T> context, IPipe<ConsumeContext<T>> next)
    {
        var tenant = context.Headers.Get<string>("Tenant-From-Send");
        if (!string.IsNullOrEmpty(tenant))
        {
            _tenant.TenantId = tenant;
        }

        return next.Send(context);
    }
}

// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseConsumeFilter(typeof(TenantConsumeFilter<>), ctx);
        cfg.ConfigureEndpoints(ctx);
    });
});
```

### Filters with custom types

```C#
// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseFilter(new TenantConsumeFilter<OrderCreated>());
        cfg.ConfigureEndpoints(ctx);
    });
});
```

### Filters not linked to a type

```C#
// Создание фильтра
public class NonTypeFilter : IFilter<ConsumeContext>
{
    public void Probe(ProbeContext context)
    {
        throw new NotImplementedException();
    }

    public Task Send(ConsumeContext context, IPipe<ConsumeContext> next)
    {
        context.ReceiveContext.TransportHeaders.TryGetHeader("Tenant-From-Send", out var tenant);
        Console.WriteLine("Hello from Non Type Filter middleware");
        return next.Send(context);
    }
}

// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UseFilter(new NonTypeFilter());
        cfg.ConfigureEndpoints(ctx);
    });
});
```

### Strongly-typed filters

```C#
// Создание фильтра
public class TenantPublishEmailFilter : IFilter<PublishContext<Email>>
{
    public void Probe(ProbeContext context)
    {
        throw new NotImplementedException();
    }

    public Task Send(PublishContext<Email> context, IPipe<PublishContext<Email>> next)
    {
        Console.WriteLine("This is an email filter");
        return next.Send(context);
    }
}

// Регистрация фильтра
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UsePublishFilter<TenantPublishEmailFilter>(ctx);
        cfg.ConfigureEndpoints(ctx);
    });
});

// Также, возможно заставить generic TenantPublishFilter<T> принимать сообщения только определенного типа
builder.Services.AddMassTransit(bus =>
{
    bus.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.UsePublishFilter(typeof(TenantPublishFilter<>), ctx, msgTypeFilter => msgTypeFilter.Include<Email>());
        cfg.ConfigureEndpoints(ctx);
    });
});
```

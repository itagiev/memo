## Tracing

Путь от запроса до ответа

### Подключение

```C#
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            // Инструментарии, помогает трейсить сервисы внутри приложения
            // Использовать внутренние инструменты .net (Activity)
            .AddAspNetCoreInstrumentation()
            .AddGrpcClientInstrumentation()
            .AddHttpClientInstrumentation()
            .AddNpgsql()
            .AddRedisInstrumentation()
            // Экспорт в jaeger
            // .AddOtlpExporter(options =>
            //     options.Endpoint = new Uri(builder.Configuration.GetValue<string>("Jaeger")!));
            // Экспорт в collector
            .AddOtlpExporter(options =>
                options.Endpoint = > otlpEndpoint);
    });
```

### Расширить трейс данными, для более удобной отладки

```C#
Activity.Current.EnrichWithClient(newClient);

public static Activity? EnrichWithClient(this Activity? activity, Client client)
{
    activity?.SetTag("client.id", client.Id);
    activity?.SetTag("client.membership", client.Membership.ToString());
    return activity;
}
```
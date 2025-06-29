## Metrics

Сбор показателей по неким агрегированным данным

### Подключение

```C#
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        metrics.AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddMeter("Microsoft.AspNetCore.Hosting")
            .AddMeter("Microsoft.AspNetCore.Server.Kestrel")
            .AddMeter(ApplicationDiagnostics.Meter.Name);

        metrics
            .AddConsoleExporter()
            // Это если prometheus, должны сам соскребать данные
            // .AddPrometheusExporter();
            // Экспорт в collector
            .AddOtlpExporter(options =>
                options.Endpoint = > otlpEndpoint);
    })
```

```C#
// Выводим api для prometheus (prometheus сам будет собирать метрики)
// только, если у нас нет коллектора, который будет делать это за нас
app.UseOpenTelemetryPrometheusScrapingEndpoint();
```

```C#
// Пример метрики для подсчета кол-ва созданных клиентов
public static readonly Meter Meter = new Meter("Clients.Api");

public static Counter<long> ClientsCreatedCounter = Meter.CreateCounter<long>("clients.created");

ApplicationDiagnostics.ClientsCreatedCounter.Add(1);
```

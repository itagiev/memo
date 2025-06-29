## Подключение

### Console app
```C#
public static class ApplicationDiagnostics
{
    public static readonly ActivitySource ActivitySource = new("Console.Tool.Diagnostics");
}

// Конфигурация сервиса
using var traceProvider = OpenTelemetry.Sdk.CreateTracerProviderBuilder()
    .SetResourceBuilder(
        ResourceBuilder.CreateDefault()
            .AddService("ConsoleTool")
    )
    // Если мы создали новый activity source, нужно его зарегестрировать,
    // иначел otel не будет следить за ним
    .AddSource(ApplicationDiagnostics.ActivitySource.Name)
    .AddConsoleExporter()
    .Build();

// Создается новый span
using var activity = ApplicationDiagnostics.ActivitySource.StartActivity("Do Work");
```

### ASP.NET

```C#
const string serviceName = "Clients.Api";
var otlpEndpoint = new Uri(builder.Configuration.GetValue<string>("OTLP_Endpoint")!);

builder.Services.AddOpenTelemetry()
    .ConfigureResource(resource =>
    {
        resource
            .AddService(serviceName)
            .AddAttributes(new[]
            {
                new KeyValuePair<string, object>("service.version",
                    Assembly.GetExecutingAssembly().GetName().Version!.ToString())
            });
    })
    // Трейсинг
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
    })
    // Метрики
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
    // Логирование
    .WithLogging(logging =>
    {
        logging.AddOtlpExporter(options =>
        {
            options.Endpoint = otlpEndpoint;
        });
    },
    options =>
    {
        options.IncludeFormattedMessage = true;
    });

// Выводим api для prometheus (prometheus сам будет собирать метрики)
// только, если у нас нет коллектора, который будет делать это за нас
app.UseOpenTelemetryPrometheusScrapingEndpoint();
```

## Работа с OpenTelemetry

- [Tracing](Tracing.md)
- [Metrics](Metrics.md)
- [Logs](Logs.md)
- [Baggage](Baggage.md)
- [Span processors](SpanProcessors.md)
- [Sampling](Sampling.md)

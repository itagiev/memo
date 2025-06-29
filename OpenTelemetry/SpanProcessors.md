## Span processors

Что-то вроде middleware для tracing.

### Использование

#### Base processor

```C#
public class BaggageProcessor : BaseProcessor<Activity>
{
    public override void OnStart(Activity data)
    {
        base.OnStart(data);
    }

    public override void OnEnd(Activity data)
    {
        // Тут осторожно, можно случайно залогировать чувствительную информацию пользователя
        foreach (var item in Baggage.Current)
        {
            if (item.Key.StartsWith("clients."))
            {
                data.SetTag(item.Key, item.Value);
            }
        }

        base.OnEnd(data);
    }
}

builder.Services.ConfigureOpenTelemetryTracerProvider(options =>
{
    options.AddProcessor(new BaggageProcessor());
});
```

Таким образом не нужно в коде, в ручную устанавливать эти параметры.
Также избавляемся от проверок на null и от using (Dispose)

#### Создание span-ов в ручную

```C#
// Создаем activity source
public static class ApplicationDiagnostics
{
    public static readonly ActivitySource ActivitySource = new("RiskEvaluator.Application");
}

// Регистрируем его
.WithTracing(tracing =>
    tracing.AddSource(ApplicationDiagnostics.ActivitySource.Name));

// Использование
using var activity = ApplicationDiagnostics.ActivitySource.StartActivity("Evaluate Email Rule");
activity?.SetTag("request.email", request.Email);
```

#### Span events

```C#
Activity.Current?.AddEvent(new ActivityEvent(
    name: "RiskResult",
    tags: [
        new("risk.score", score),
        new("risk.level", level)
    ]));
```

Можно использовать это за место логов, тогда в span мы увидем секцию лог с параметрами, которые передаем.

#### Errors and Exceptions

```C#
try
{
    // ...
}
catch (Exception exception)
{
    Activity.Current?.SetStatus(Status.Error);
    Activity.Current?.RecordException(exception);

    return Task.FromResult(new RiskEvaluationReply()
    {
        RiskLevel = RiskLevel.High,
    });
}
```

#### Span links

Пример, обновление данных пакетами. Создается новый trace с сылками на другие trace, которые и составляют этот пакет.

```C#
List<ActivityLink> activityLinks = new();
/*
 * Снчала нужно будет вытачить эти линки с помощью propagation context
 */
var parentContext = RabbitMqDiagnostics.Propagator.Extract(
    default,
    ea.BasicProperties,
    ExtractTraceContextFromBasicProperties);

activityLinks.Add(new ActivityLink(parentContext.ActivityContext));

using var activity = ApplicationDiagnostics.ActivitySource.StartActivity(
    "Report Process",
    ActivityKind.Internal,
    new ActivityContext(),
    links: activityLinks);

activityLinks.Clear();
```

## Baggage

Контекстная информация, которая передается между сервисами.

К примеру context propagation в брокере сообщений.

#### Publisher

```C#
Baggage.SetBaggage("client.id", client.Id.ToString());

const string operation = "publish";
var eventType = @event!.GetType().Name;
// Start an activity with a name following the semantic convention of the OpenTelemetry messaging specification.
// https://github.com/open-telemetry/semantic-conventions/blob/main/docs/messaging/messaging-spans.md
var activityName = $"{eventType} {operation}";
using var activity = RabbitMqDiagnostics.ActivitySource.StartActivity(activityName, ActivityKind.Producer);

ActivityContext contextToInject = default;

if (activity != null)
{
    contextToInject = activity.Context;
}
else if (Activity.Current != null)
{
    contextToInject = Activity.Current.Context;
}

var properties = channel.CreateBasicProperties();
properties.DeliveryMode = 2;

RabbitMqDiagnostics.Propagator.Inject(
    new PropagationContext(contextToInject, Baggage.Current),
    properties,
    InjectTraceContextIntoBasicProperties);

SetActivityContext(activity, eventType, operation);

private void SetActivityContext(Activity? activity, string eventType, string operation)
{
    if (activity is null) return;

    // These tags are added demonstrating the semantic conventions of the OpenTelemetry messaging specification
    // https://github.com/open-telemetry/semantic-conventions/blob/main/docs/messaging/messaging-spans.md
    activity.SetTag("messaging.system", "rabbitmq");
    activity.SetTag("messaging.destination_kind", "exchange");
    activity.SetTag("messaging.operation", operation);
    activity.SetTag("messaging.destination.name", eventType);
}

private void InjectTraceContextIntoBasicProperties(IBasicProperties props, string key, string value)
{
    props.Headers ??= new Dictionary<string, object>();
    props.Headers[key] = value;
}
```

#### Consumer

```C#
var parentContext = RabbitMqDiagnostics.Propagator.Extract(default,
    @event.BasicProperties,
    ExtractTraceContextFromBasicProperties);

Baggage.Current = parentContext.Baggage;

// Start an activity with a name following the semantic convention of the OpenTelemetry messaging specification.
// https://github.com/open-telemetry/semantic-conventions/blob/main/docs/messaging/messaging-spans.md
const string operation = "process";
var activityName = $"{@event.RoutingKey} {operation}";

using var activity = RabbitMqDiagnostics.ActivitySource.StartActivity(activityName, ActivityKind.Consumer,
    parentContext.ActivityContext);

SetActivityContext(activity, @event.RoutingKey, operation);

var body = @event.Body.ToArray();
var message = Encoding.UTF8.GetString(body);
var data = JsonSerializer.Deserialize<T>(message);

activity?.SetTag("message", message); // DEMO ONLY
activity?.SetTag("client.id", Baggage.Current.GetBaggage("client.id"));

private static void SetActivityContext(Activity? activity, string eventName, string operation)
{
    if (activity is null) return;
    // These tags are added demonstrating the semantic conventions of the OpenTelemetry messaging specification
    // https://github.com/open-telemetry/semantic-conventions/blob/main/docs/messaging/messaging-spans.md
    activity.SetTag("messaging.system", "rabbitmq");
    activity.SetTag("messaging.destination_kind", "queue");
    activity.SetTag("messaging.operation", operation);
    activity.SetTag("messaging.destination.name", eventName);
}
private IEnumerable<string> ExtractTraceContextFromBasicProperties(IBasicProperties props, string key)
{
    if (!props.Headers.TryGetValue(key, out var value)) return [];

    var bytes = value as byte[];
    return [Encoding.UTF8.GetString(bytes)];
}
```
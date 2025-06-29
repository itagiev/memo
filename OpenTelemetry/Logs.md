## Logs

### Подключение

```C#
builder.Services.AddOpenTelemetry()
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
```

```C#
_logger.LogInformation("New user {Email} added.", user.Email);
```
## Setup

### Установка

```sh
dotnet add package MassTransit.RabbitMQ
```

### Подключение

```C#
builder.Services.AddMassTransit(options =>
{
    options.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("localhost", "/", hostCfg =>
        {
            hostCfg.Username("guest");
            hostCfg.Password("guest");
        });

        cfg.ConfigureEndpoints(context);
    });
});
```

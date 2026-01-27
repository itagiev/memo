## Logging in .NET

- [Уровни](./LogLevel.md)

### Event Id

```C#
public class LogEvents
{
    public const int HomePageHitEvent = 123;
}

app.MapGet("/", (ILogger<Program> logger) =>
{
    logger.LogInformation(LogEvents.HomePageHitEvent, "Home page");
    return 0;
});

// info: Program[123]
// Home page
```

## Нетестируемый код

К примеру `ILogger<T>` - некоторые используемые объекты являются внутренними (internal), а вызываемые методы являются расширениями.
С начала создается адаптер

```C#
public interface ILoggerAdapter<out TCategoryName>
{
    void LogInformation(string? message, params object?[] args);

    void LogError(Exception? exception, string? message, params object?[] args);
}

public class LoggerAdapter<out TCategoryName> : ILoggerAdapter<TCategoryName>
{
    private readonly ILogger<TCategoryName> _logger;

    public LoggerAdapter(ILogger<TCategoryName> logger)
    {
        _logger = logger;
    }

    public void LogInformation(string? message, params object?[] args)
    {
        _logger.LogInformation(message, args);
    }

    public void LogError(Exception? exception, string? message, params object?[] args)
    {
        _logger.LogError(exception, message, args);
    }
}
```

Регистрация адаптера

```C#
builder.Services.AddTransient(typeof(ILoggerAdapter<>), typeof(LoggerAdapter<>));
```

Замена стандартного логгера адаптером

```C#
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly ILoggerAdapter<UserService> _logger;

    public UserService(IUserRepository userRepository,
        ILoggerAdapter<UserService> logger)
    {
        _userRepository = userRepository;
        _logger = logger;
    }

    public async Task<IEnumerable<User>> GetAllAsync()
    {
        _logger.LogInformation("Retrieving all users");
        var stopWatch = Stopwatch.StartNew();
        try
        {
            return await _userRepository.GetAllAsync();
        }
        catch (Exception e)
        {
            _logger.LogError(e, "Something went wrong while retrieving all users");
            throw;
        }
        finally
        {
            stopWatch.Stop();
            _logger.LogInformation("All users retrieved in {0}ms", stopWatch.ElapsedMilliseconds);
        }
    }
}
```

Тестирование

```C#
// Mock логгера
private readonly ILoggerAdapter<UserService> _logger = Substitute.For<ILoggerAdapter<UserService>>();
// Assert
_logger.Received(1).LogInformation(Arg.Is("Retrieving all users"));
_logger.Received(1).LogInformation(Arg.Is("All users retrieved in {0}ms"), Arg.Any<long>());
// Пример с exception
var sqliteException = new SqliteException("Something went wrong", 500);
_logger.Received(1).LogError(Arg.Is(sqliteException), Arg.Is("Something went wrong while retrieving all users"));
```

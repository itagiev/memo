## Пример работы с тестовыми контейнерами

Библиотека `Testcontainers`

```C#
public class CustomerApiFactory : WebApplicationFactory<IWebMarker>, IAsyncLifetime
{
    // Создание контейнера
    private readonly PostgreSqlContainer _postgresContainer = new PostgreSqlBuilder()
        .WithImage("postgres:16.3-alpine3.20")
        .WithUsername("postgres")
        .WithPassword("postgres")
        .Build();

    // Из интерфейса IAsyncLifetime
    public async Task InitializeAsync()
    {
        // Запуска контейнера
        await _postgresContainer.StartAsync();
        // Получить внешний порт контейнера
        int port = _postgresContainer.GetMappedPublicPort();

        // Так как приложение берет данные, такие как строки подключения к БД, из конфигурации,
        // нужно установить эти переменные
        Environment.SetEnvironmentVariable("OpenIddict__ClientSecret", "secret");
        Environment.SetEnvironmentVariable("OpenIddict__Issuer", "http://localhost:5110/");
        Environment.SetEnvironmentVariable("Data__ApplicationDbContext__Provider", "Postgres");
        Environment.SetEnvironmentVariable("Data__QuartzDbContext__Provider", "Postgres");
        Environment.SetEnvironmentVariable("Data__ApplicationDbContext__ConnectionString",
            $"Host=localhost;Port={port};Database=renna_estore_accounts;Username=postgres;Password=postgres");
        Environment.SetEnvironmentVariable("Data__QuartzDbContext__ConnectionString",
            $"Host=localhost;Port={port};Database=renna_estore_accounts_scheduler;Username=postgres;Password=postgres");
    }

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // Можно очистить логгинг, так как в тестах этого нет
        builder.ConfigureLogging(logging =>
        {
            logging.ClearProviders();
        });

        // При желании можно удалить какой-то сервис, добавленый из основного приложения
        // и заменить его своим
        // Тут пример с базой данных, это было бы полезно, если бы строка подключения была вшита в код
        builder.ConfigureServices(services =>
        {
            //var appDbContextDescriptor = services.SingleOrDefault(
            //    d => d.ServiceType ==
            //        typeof(IDbContextOptionsConfiguration<ApplicationDbContext>));

            //services.Remove(appDbContextDescriptor!);

            //services.AddApplicationDbContext(true, "Postgres",
            //    $"Host=localhost;Port={_postgresContainer.GetMappedPublicPort()};Database=renna_estore_accounts;Username=postgres;Password=postgres");
        });
    }

    // Из интерфейса IAsyncLifetime
    async Task IAsyncLifetime.DisposeAsync()
    {
        // Остановка контейнера, контейнер сам удаляется
        await _postgresContainer.StopAsync();
    }
}
```
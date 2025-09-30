## Работа с интеграционными тестами

The 5 integration testing steps:
1. Setup
2. Dependency mocking (API)
3. Execution
4. Assertion
5. Cleanup

### WebApplicationFactory

Для того, чтобы использовать наш web api, можно эмуливать его работу с помощью `WebApplicationFactory<TEntryPoint>`.
И вызывать нужные api методы с помощью `HttpClient`

```C#
public class CustomersControllerTests : IClassFixture<WebApplicationFactory<IWebMarker>>
{
    private readonly HttpClient _httpClient;

    public CustomersControllerTests(WebApplicationFactory<IWebMarker> _webAppFactory)
    {
        _httpClient = _webAppFactory.CreateClient();
    }

    [Fact]
    public async Task FindCustomerById_ReturnsNotFound_WhenCustomerDoesntExist()
    {
        // ...
        var response = await _httpClient.GetAsync($"api/customers/{int.MinValue}");
        // ...
    }
}
```

Для использовании данного имени нужно установить библиотеку Microsoft.AspNetCore.Mvc.Testing

### Fake data generator

Чтобы генерировать случайные, но реалистичные данные, можно воспользоваться библиотекой Bogus

```C#
private readonly Faker<AddCustomerCommand> _addCustomerCommandGenerator = new Faker<AddCustomerCommand>()
    .RuleFor(x => x.CustomerUUID, faker => faker.Random.Guid())
    .RuleFor(x => x.TabNumber, faker => $"{faker.Random.String2(2)}-{faker.Random.Number(10000, 999999)}")
    .RuleFor(x => x.Email, faker => faker.Person.Email)
    .RuleFor(x => x.FullName, faker => faker.Person.FullName);

var command = _addCustomerCommandGenerator.Generate();
```
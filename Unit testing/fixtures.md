
### Class Fixture

Так как для каждого теста создается свой экземпляр тест класса, то и данные для каждого теста будут свои.
Чтобы расшарить данные можно применить интерфейс `IClassFixture<T>`

Пример

```C#
public class CustomerStoreTests : IClassFixture<CustomerMock>
{
    private readonly CustomerMock _customerMock;

    public CustomerStoreTests(CustomerMock customerMock)
    {
        _customerMock = customerMock;
    }

    // Теперь _customerMock для каждого теста [Fact]/[Theory] будет один и тот же экземпляр
}

public class CustomerMock : IDisposable
{
    // ...
}
```

### Collection Fixture

```C#
using Xunit.Abstractions;

namespace UnitTests;

public class SampleClassFixture : IDisposable
{
    public string Id { get; } = Guid.NewGuid().ToString();

    public void Dispose() { }
}

[CollectionDefinition("Sample class collection")]
public class SampleClassCollectionFixture : ICollectionFixture<SampleClassFixture>;

[Collection("Sample class collection")]
public class CollectionFixturesBehaviorTests
{
    private readonly SampleClassFixture _classFixture;
    private readonly ITestOutputHelper _testOutputHelper;

    public CollectionFixturesBehaviorTests(SampleClassFixture classFixture,
        ITestOutputHelper testOutputHelper)
    {
        _classFixture = classFixture;
        _testOutputHelper = testOutputHelper;
    }

    [Fact]
    public void ExampleTest1()
    {
        _testOutputHelper.WriteLine(_classFixture.Id);
    }

    [Fact]
    public void ExampleTest2()
    {
        _testOutputHelper.WriteLine(_classFixture.Id);
    }
}

[Collection("Sample class collection")]
public class CollectionFixturesBehaviorTests2
{
    private readonly SampleClassFixture _classFixture;
    private readonly ITestOutputHelper _testOutputHelper;

    public CollectionFixturesBehaviorTests2(SampleClassFixture classFixture,
        ITestOutputHelper testOutputHelper)
    {
        _classFixture = classFixture;
        _testOutputHelper = testOutputHelper;
    }

    [Fact]
    public void ExampleTest1()
    {
        _testOutputHelper.WriteLine(_classFixture.Id);
    }

    [Fact]
    public void ExampleTest2()
    {
        _testOutputHelper.WriteLine(_classFixture.Id);
    }
}
```
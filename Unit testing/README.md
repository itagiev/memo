## Написание Unit тестов

[Fact]

### Naming

Наименование проектов:
- ClassLibrary.UnitTests

Наименование классов:
- CalculatorTests

Наименование тестов:

Имя метода -> Что должен делать (Should) -> При каких условиях (When)

`void Add_ShouldAddTwoNumbers_WhenTwoNumbersAreIntegers()`

### Arrange -> Act -> Assert

```C#
[Fact]
public void NormalizeTabNumber_ShouldReturnNormalizedTabNumber_WhenTabNumberIsString()
{
    // Arrange
    var lookupNormalizer = new UpperInvariantLookupNormalizer();
    string tabNumber = "asd-32134";
    // Act
    var normalizedTabNumber = lookupNormalizer.NormalizeTabNumber(tabNumber);
    // Assert
    Assert.Equal("ASD-32134", normalizedTabNumber);
}
```

### Test execution model (context)

```C#
public class UpperInvariantLookupNormalizerTests
{
    // В данном случае _lookupNormalizer - system under test
    // ВАЖНО: для каждого теста будет создан свой экзмепляр
    private readonly UpperInvariantLookupNormalizer _lookupNormalizer = new();

    [Fact]
    public void NormalizeTabNumber_ShouldReturnNormalizedTabNumber_WhenTabNumberIsString()
    {
        // Arrange
        string tabNumber = "asd-32134";
        // Act
        var normalizedTabNumber = _lookupNormalizer.NormalizeTabNumber(tabNumber);
        // Assert
        Assert.Equal("ASD-32134", normalizedTabNumber);
    }

    [Fact]
    public void NormalizeTabNumber_ShouldReturnNull_WhenTabNumberIsNull()
    {
        // Arrange
        string? tabNumber = null;
        // Act
        var normalizedTabNumber = _lookupNormalizer.NormalizeTabNumber(tabNumber);
        // Assert
        Assert.Null(normalizedTabNumber);
    }
}
```

### Setup and teardown

- Для нормального кода реализуем `IDisposable`
- Для асинхронного кода - `IAsyncLifetime`

### Parameterizing

```C#
[Theory]
[InlineData("ASD-32134", "asd-32134")]
[InlineData("322", "322")]
public void NormalizeTabNumber_ShouldReturnNormalizedTabNumber_WhenTabNumberIsString(
    string expected, string tabNumber)
{
    // Act
    var normalizedTabNumber = _lookupNormalizer.NormalizeTabNumber(tabNumber);

    // Assert
    Assert.Equal(expected, normalizedTabNumber);
}
```

### Ignoring tests

```C#
[Fact(Skip = "This breaks in CI")]
[Theory(Skip = "This breaks in CI")]
[InlineData("ASD-32134", "asd-32134", Skip = "This breaks in CI")]
```

### Fluent assertion

#### Strings

```C#
string name = "John";
name.Should().Be("John");
name.Should().NotBeEmpty();
name.Should().StartWith("J");
name.Should().Contain("oh");
```

#### Numbers / Dates

```C#
int age = 21;
age.Should().Be(22);
age.Should().BePositive();
age.Should().BeGreaterThanOrEqualTo(21);
age.Should().BeLessThan(60);
age.Should().BeInRange(21, 60);
```

#### Objects

```C#
var user = new User { Age = 21, Name = "John" };
var expected = new User { Age = 21, Name = "John" };
// Дефолтное сравнение как Equals (сравнение ссылок)
user.Should().Be(expected);
// Сравнение значений
user.Should().BeEquivalentTo(expected);
```

#### Enumerables

```C#
var expected = new User { Age = 21, Name = "John" };
var users = _users.ToArray();
// By value
users.Should().ContainEquivalentOf(expected);
users.Should().HaveCount(3);
users.Should().Contain(x => x.Name == "John" && x.Age > 10);
```

#### Methods

```C#
public class User
{
    public int Age { get; set; }
    public string? Name { get; set; }
    public bool IsOld()
    {
        throw new NotImplementedException("Not implemented yet.");
    }
}

var user = new User { Age = 21, Name = "John" };
Func<bool> act = () => user.IsOld();
act.Should().Throw<NotImplementedException>().WithMessage("Not implemented yet.");
```

#### Events

```C#
var values = new Values();
var monitor = values.Monitor();
values.RaiseSampleEvent();
monitor.Should().Raise("ValuesEvent");
```

#### Internal fields/methods

```C#
// Создать файл .cs в проекте в котором находятся необходимые internal поля
// И вызвать этот метод для определенного домена
[assembly: InternalsVisibleTo("UnitTests")]
```
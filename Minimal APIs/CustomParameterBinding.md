#### Custom Parameter Binding

Допустим у нас есть класс Point \
Чтобы получить его из Query (?p=1.55,2.32), нужно реализовать метод TryParse \
Чтобы получить из post запроса, например из body в виде raw text, нужно реализовать метод BindAsync

`Point.cs`

```C#
public class Point
{
    public required double Latitude { get; init; }

    public required double Longitude { get; init; }

    public static bool TryParse(string? value, out Point? point)
    {
        try
        {
            var valueAsSpan = value.AsSpan();
            Span<double> data = stackalloc double[2];
            int i = 0;

            foreach (var range in valueAsSpan.Split(','))
            {
                data[i++] = double.Parse(valueAsSpan[range], CultureInfo.InvariantCulture);
            }

            point = new Point
            {
                Latitude = data[0],
                Longitude = data[1]
            };

            return true;
        }
        catch
        {
            point = null;
            return false;
        }
    }

    public static async ValueTask<Point?> BindAsync(HttpContext context, ParameterInfo parameterInfo)
    {
        var value = await new StreamReader(context.Request.Body).ReadToEndAsync();

        if (TryParse(value, out var point))
        {
            return point;
        }

        return null;
    }
}
```

`Program.cs`

```C#
// ?p=1.55,2.32
app.MapGet("get-point", ([FromQuery(Name = "p")] Point point) => point);

// Post raw text "1.55,2.32"
app.MapPost("post-point", (Point point) => point);
```
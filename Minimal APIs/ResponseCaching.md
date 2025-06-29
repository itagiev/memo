#### Response Caching Middleware in ASP.NET Core

```C#
// Конфигурация
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCaching();

var app = builder.Build();

app.UseHttpsRedirection();

// UseCors must be called before UseResponseCaching when using CORS middleware.
//app.UseCors

app.UseResponseCaching();

app.Use(async (context, next) =>
{
    // Cache-Control: Caches cacheable responses for up to 10 seconds.
    context.Response.GetTypedHeaders().CacheControl =
        new Microsoft.Net.Http.Headers.CacheControlHeaderValue()
        {
            Public = true,
            MaxAge = TimeSpan.FromSeconds(10)
        };
    // Vary: Configures the middleware to serve a cached response
    // only if the Accept-Encoding header of subsequent requests matches that of the original request.
    context.Response.Headers[Microsoft.Net.Http.Headers.HeaderNames.Vary] =
        new string[] { "Accept-Encoding" };

    await next();
});

// Map endpoints...

app.Run();
```

```C#
// Что-бы варьировать cache в зависимости от параметров запроса, в методе endpoint-а нужно задать
if (context.Features.Get<IResponseCachingFeature>() is IResponseCachingFeature feature)
{
    feature.VaryByQueryKeys = ["title", "year", "sortBy", "page", "pageSize"];
}
```

[Learn Microsoft](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/middleware?view=aspnetcore-9.0)
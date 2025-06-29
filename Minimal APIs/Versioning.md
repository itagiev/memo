#### Версионирование

```C#
builder.Services
    .AddApiVersioning(options =>
    {
        options.ApiVersionReader = new MediaTypeApiVersionReader("api-version");
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.AssumeDefaultVersionWhenUnspecified = true;
        // reporting api versions will return the headers
        // "api-supported-versions" and "api-deprecated-versions"
        options.ReportApiVersions = true;
    })
    .AddApiExplorer();
// ...
var app = builder.Build();
app.UseApiVersionSet();
```

```C#
public static class ApiVersioning
{
    public static ApiVersionSet VersionSet { get; private set; } = null!;

    public static IEndpointRouteBuilder UseApiVersionSet(this IEndpointRouteBuilder app)
    {
        VersionSet = app.NewApiVersionSet()
            .HasApiVersion(new ApiVersion(1.0d))
            .HasApiVersion(new ApiVersion(2.0d))
            .ReportApiVersions()
            .Build();

        return app;
    }
}
```

```C#
var group = app.MapGroup(ApiEndpoints.Movies.Prefix)
    .WithApiVersionSet(ApiVersioning.VersionSet);

group.MapPost(ApiEndpoints.Movies.Create, CreateMovieAsync)
    .WithName(nameof(CreateMovieAsync))
    .Produces<MovieResponse>(StatusCodes.Status201Created)
    .Produces<ValidationFailureResponse>(StatusCodes.Status400BadRequest)
    .RequireAuthorization(AuthConstants.TrustedMemberPolicyName)
    .MapToApiVersion(1.0d);

group.MapGet(ApiEndpoints.Movies.Get, GetMovieAsync)
    .WithName(nameof(GetMovieAsync))
    .Produces<MovieResponse>(StatusCodes.Status200OK)
    .Produces(StatusCodes.Status404NotFound)
    .MapToApiVersion(1.0d);

group.MapGet(ApiEndpoints.Movies.Get, GetMovieV2Async)
    .WithName(nameof(GetMovieV2Async))
    .Produces<MovieResponse>(StatusCodes.Status200OK)
    .Produces(StatusCodes.Status404NotFound)
    .MapToApiVersion(2.0d);
```
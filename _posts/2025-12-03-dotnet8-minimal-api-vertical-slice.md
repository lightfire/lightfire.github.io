---
title: "Building Clean and Scalable APIs with .NET 8 Minimal API and Vertical Slice Architecture"
description: "A lean template that combines Minimal API, MediatR, FluentValidation, and Vertical Slice for fast, testable services."
categories: [Backend, .NET]
tags: [dotnet, minimal-api, vertical-slice, mediatr, fluentvalidation, cqrs]
---

Modern .NET 8 simplifies API development with Minimal API while staying production-ready. By pairing it with Vertical Slice Architecture, MediatR, and FluentValidation, each feature lives in its own folder, stays testable, and avoids bloated service layers.

## Why Vertical Slice over classic layers?
- Each feature is self-contained (request, validation, handler, endpoint).
- No swollen service layer; changes are localized.
- Natural CQRS: commands and queries have separate handlers.
- Easier testing: validators, handlers, and endpoints can be tested in isolation.

Folder sketch:
```
VerticalApi
 └── Features
      └── Users
          └── CreateUser
                CreateUserRequest.cs
                CreateUserValidator.cs
                CreateUserHandler.cs
                CreateUserEndpoint.cs
 └── Program.cs
```

## Project setup
```bash
dotnet new webapi -n VerticalApi
cd VerticalApi
dotnet add package MediatR
dotnet add package FluentValidation
```

## 1) Request
```csharp
public record CreateUserRequest(string Email, string Password) : IRequest<IResult>;
```

## 2) Validation (FluentValidation)
```csharp
using FluentValidation;

public class CreateUserValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress();

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(6);
    }
}
```

## 3) Handler (MediatR)
```csharp
using MediatR;

public class CreateUserHandler : IRequestHandler<CreateUserRequest, IResult>
{
    public async Task<IResult> Handle(CreateUserRequest request, CancellationToken ct)
    {
        // Simulated persistence
        await Task.Delay(50, ct);

        return Results.Ok(new
        {
            message = "User successfully created",
            user = new { request.Email }
        });
    }
}
```

## 4) Endpoint (Minimal API extension)
```csharp
public static class CreateUserEndpoint
{
    public static void MapCreateUserEndpoint(this IEndpointRouteBuilder app)
    {
        app.MapPost("/users", async (
            CreateUserRequest req,
            IMediator mediator,
            IValidator<CreateUserRequest> validator) =>
        {
            var validation = await validator.ValidateAsync(req);
            if (!validation.IsValid)
                return Results.BadRequest(validation.Errors);

            return await mediator.Send(req);
        })
        .WithName("CreateUser")
        .WithOpenApi();
    }
}
```

## 5) Program.cs
```csharp
using FluentValidation;
using MediatR;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddMediatR(typeof(Program));
builder.Services.AddValidatorsFromAssembly(typeof(Program).Assembly);

var app = builder.Build();
app.MapCreateUserEndpoint();
app.Run();
```

## Testing the API
Request:
```http
POST /users
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "secret123"
}
```
Response:
```json
{
  "message": "User successfully created",
  "user": { "email": "test@example.com" }
}
```

## Why this is production-friendly
1. Feature-based development: each slice is independent and easy to evolve.
2. Controller-free: endpoints stay close to their handlers and validation.
3. CQRS-ready: commands and queries can stay separate without ceremony.
4. Testable: validator tests, handler tests, and endpoint tests are straightforward.
5. Lean but clean: avoids heavy layers while keeping responsibilities clear.

## Next steps you can add
- Persistence: EF Core repositories with PostgreSQL or SQL Server.
- Auth: JWT bearer authentication and authorization policies.
- Observability: Serilog + OpenTelemetry for logs, metrics, and traces.
- Resilience: exception middleware, validation problem details, health checks.
- Separation: dedicated Query handlers alongside Command handlers for read models.

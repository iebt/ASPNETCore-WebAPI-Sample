# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ASP.NET Core 6.0 Web API sample demonstrating RESTful API design with HATEOAS (Hypermedia as the Engine of Application State), API versioning, and Swagger documentation. The API manages a simple Food items resource with full CRUD operations.

## Build and Run Commands

### Build
```bash
dotnet build --configuration Release
```

### Run (Development)
```bash
dotnet run --project SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj
```

The API runs on `http://localhost:29435` by default.

### Access Swagger UI
Navigate to `http://localhost:29435/swagger` when running in Development mode.

## Architecture

### Project Structure
- **Controllers/**: API endpoints organized by version (v1/, v2/)
- **Repositories/**: Data access layer with `IFoodRepository` interface and `FoodSqlRepository` implementation
- **Services/**: Business logic services (e.g., `ISeedDataService` for test data)
- **Entities/**: Database entities (e.g., `FoodEntity`)
- **Dtos/**: Data Transfer Objects (`FoodDto`, `FoodCreateDto`, `FoodUpdateDto`)
- **Models/**: Domain models (e.g., `QueryParameters`, `LinkDto`)
- **MappingProfiles/**: AutoMapper configuration (e.g., `FoodMappings`)
- **Helpers/**: Extension methods for cross-cutting concerns

### Key Architectural Patterns

**Dependency Injection**: Services are registered in `Program.cs`:
- `IFoodRepository` → `FoodSqlRepository` (Scoped)
- `ISeedDataService` → `SeedDataService` (Singleton)
- AutoMapper with `FoodMappings` profile

**Repository Pattern**: Data access is abstracted through `IFoodRepository` interface. The implementation uses Entity Framework Core with an in-memory database (`UseInMemoryDatabase("FoodDatabase")`).

**API Versioning**: Configured via `VersioningExtension.AddVersioning()`:
- URL segment versioning: `/api/v1/foods`, `/api/v2/foods`
- Header versioning: `x-api-version` header
- Media type versioning: `x-api-version` in Accept/Content-Type
- Default version: 1.0
- Controllers use `[ApiVersion("1.0")]` or `[ApiVersion("2.0")]` attributes
- Routes use `[Route("api/v{version:apiVersion}/[controller]")]`

**HATEOAS Implementation**:
- v1 FoodsController includes hypermedia links in responses
- `ExpandSingleFoodItem()` adds `links` array to each resource
- `CreateLinksForCollection()` provides navigation links (self, first, last, next, previous)
- Links use named routes (e.g., `nameof(GetSingleFood)`)
- The `LinkDto` model represents hypermedia links
- `DynamicExtensions.ToDynamic()` converts DTOs to expandable objects for adding links

**Pagination**: Handled via `QueryParameters` model:
- `Page`: Current page number (default: 1)
- `PageCount`: Items per page (max: 50, default: 50)
- `OrderBy`: Sort field (default: "Name")
- Pagination metadata returned in `X-Pagination` response header
- Dynamic sorting via `System.Linq.Dynamic.Core` library

**Filtering**: QueryParameters supports text search via `Query` property. The repository searches both `Name` and `Calories` fields.

**AutoMapper**: Maps between Entities and DTOs:
- `FoodEntity` ↔ `FoodDto` / `FoodCreateDto` / `FoodUpdateDto`
- Configuration in `MappingProfiles/FoodMappings.cs`

**JSON Patch**: PATCH endpoint (`PartiallyUpdateFood`) uses `JsonPatchDocument<FoodUpdateDto>` from `Microsoft.AspNetCore.JsonPatch` for partial updates.

**CORS**: Configured via `CorsExtension.AddCustomCors("AllowAllOrigins")` - allows all origins in this sample.

**Seed Data**: In Development mode, `SeedDataExtension.SeedData()` populates the in-memory database with test food items.

### Response Patterns

**v1 API responses** include:
- `value`: Array of resources or single resource with embedded HATEOAS links
- `links`: Array of `LinkDto` objects for navigation

**Pagination metadata** (in `X-Pagination` header):
```json
{
  "totalCount": 20,
  "pageSize": 10,
  "currentPage": 1,
  "totalPages": 2
}
```

## Key Dependencies

- **Microsoft.EntityFrameworkCore.InMemory**: In-memory database provider
- **AutoMapper**: Object-to-object mapping
- **Microsoft.AspNetCore.Mvc.Versioning**: API versioning
- **Swashbuckle.AspNetCore**: Swagger/OpenAPI generation
- **Microsoft.AspNetCore.Mvc.NewtonsoftJson**: JSON PATCH support
- **System.Linq.Dynamic.Core**: Dynamic LINQ for runtime query construction

## Development Notes

**Configuration**: Settings in `appsettings.json` and `appsettings.Development.json`.

**Exception Handling**: Production mode uses `ExceptionExtension.AddProductionExceptionHandling()` for centralized error handling.

**Swagger Configuration**: `ConfigureSwaggerOptions` class implements `IConfigureOptions<SwaggerGenOptions>` to configure Swagger per API version.

**Routing**: Lowercase URLs enforced via `AddRouting(options => options.LowercaseUrls = true)`.

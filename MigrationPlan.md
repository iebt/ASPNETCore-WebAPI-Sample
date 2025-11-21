# .NET 9 Migration Plan

## 1. Current State

### Projects and Target Frameworks
- **SampleWebApiAspNetCore.csproj**: `net6.0`
  - Location: `/SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj`
  - Type: ASP.NET Core Web API

### Current Dependencies
- **AutoMapper**: 11.0.1
- **AutoMapper.Extensions.Microsoft.DependencyInjection**: 11.0.0
- **Microsoft.AspNetCore.Mvc.NewtonsoftJson**: 6.0.8
- **Microsoft.AspNetCore.Mvc.Versioning**: 5.0.0 (deprecated)
- **Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer**: 5.0.0 (deprecated)
- **Microsoft.EntityFrameworkCore.InMemory**: 6.0.8
- **Swashbuckle.AspNetCore**: 6.4.0
- **System.Linq.Dynamic.Core**: 1.2.19

### SDK and Runtime
- **GitHub Actions Workflow**: .NET SDK 6.0.x
  - File: `.github/workflows/dotnetcore.yml:15`
  - Action: `actions/setup-dotnet@v2`

### Build Configuration
- No `global.json` file present
- No Docker configuration found
- Build command: `dotnet build --configuration Release`

## 2. Migration Scope

### In Scope
This migration is **strictly a framework version upgrade to .NET 9**. The following changes are included:

- Update `TargetFramework` from `net6.0` to `net9.0`
- Update NuGet packages to .NET 9-compatible versions
- Replace deprecated API versioning packages with modern equivalents
- Update GitHub Actions workflow to use .NET 9 SDK
- Address breaking changes required for compilation and runtime compatibility
- Validate build and basic functionality

### Out of Scope
The following are **explicitly excluded** from this migration:

- Code refactoring or architectural changes
- Performance optimizations
- New feature additions
- Code style or formatting changes
- Migration to Minimal APIs
- Migration from Newtonsoft.Json to System.Text.Json
- Documentation updates (beyond this plan)
- Test coverage improvements
- Security enhancements (beyond what .NET 9 provides)

## 3. Required Changes

### 3.1 Project File Updates

**SampleWebApiAspNetCore.csproj**:

1. **Target Framework** (Line 4):
   - Change: `<TargetFramework>net6.0</TargetFramework>` → `<TargetFramework>net9.0</TargetFramework>`

2. **Package Updates**:
   - **AutoMapper**: 11.0.1 → 13.0.1 (latest stable before license requirement in v14+)
   - **AutoMapper.Extensions.Microsoft.DependencyInjection**: Remove package (deprecated, no longer needed)
   - **Microsoft.AspNetCore.Mvc.NewtonsoftJson**: 6.0.8 → 9.0.0
   - **Microsoft.AspNetCore.Mvc.Versioning**: 5.0.0 → Replace with **Asp.Versioning.Mvc** 8.1.0
   - **Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer**: 5.0.0 → Replace with **Asp.Versioning.Mvc.ApiExplorer** 8.1.0
   - **Microsoft.EntityFrameworkCore.InMemory**: 6.0.8 → 9.0.0
   - **Swashbuckle.AspNetCore**: 6.4.0 → 7.2.0 (or latest 7.x compatible with .NET 9)
   - **System.Linq.Dynamic.Core**: 1.2.19 → 1.4.10 (latest stable)

### 3.2 Code Changes

**Program.cs** - API Versioning namespace changes:

The deprecated `Microsoft.AspNetCore.Mvc.Versioning` packages use different namespaces than the new `Asp.Versioning.Mvc` packages. Expected changes:

1. Update using statements:
   - Old: `using Microsoft.AspNetCore.Mvc.Versioning;`
   - New: `using Asp.Versioning;`

2. Service registration may require slight syntax adjustments (verify after package update)

**Note**: The exact code changes will be confirmed after attempting compilation with updated packages. The `Asp.Versioning` packages maintain similar API surface area to minimize code changes.

### 3.3 CI/CD Updates

**.github/workflows/dotnetcore.yml** (Line 15):

```yaml
# Current
dotnet-version: '6.0.x'

# Updated
dotnet-version: '9.0.x'
```

### 3.4 SDK Version

**Recommendation**: Add `global.json` to pin SDK version for consistency:

```json
{
  "sdk": {
    "version": "9.0.100",
    "rollForward": "latestFeature"
  }
}
```

## 4. Migration Steps

Execute the migration in the following order:

### Step 1: Prepare and Backup
1. Ensure current code compiles on .NET 6: `dotnet build --configuration Release`
2. Commit all pending changes to source control
3. Create migration branch: `git checkout -b migrate-to-dotnet9`

### Step 2: Update SDK References
1. Add `global.json` with .NET 9 SDK version (optional but recommended)
2. Update `.github/workflows/dotnetcore.yml` to use .NET 9 SDK

### Step 3: Update Project Target Framework
1. Edit `SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj`
2. Change `<TargetFramework>net6.0</TargetFramework>` to `<TargetFramework>net9.0</TargetFramework>`

### Step 4: Update NuGet Packages
1. Remove deprecated package:
   ```bash
   dotnet remove SampleWebApiAspNetCore package AutoMapper.Extensions.Microsoft.DependencyInjection
   ```

2. Replace versioning packages:
   ```bash
   dotnet remove SampleWebApiAspNetCore package Microsoft.AspNetCore.Mvc.Versioning
   dotnet remove SampleWebApiAspNetCore package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
   dotnet add SampleWebApiAspNetCore package Asp.Versioning.Mvc --version 8.1.0
   dotnet add SampleWebApiAspNetCore package Asp.Versioning.Mvc.ApiExplorer --version 8.1.0
   ```

3. Update remaining packages:
   ```bash
   dotnet add SampleWebApiAspNetCore package AutoMapper --version 13.0.1
   dotnet add SampleWebApiAspNetCore package Microsoft.AspNetCore.Mvc.NewtonsoftJson --version 9.0.0
   dotnet add SampleWebApiAspNetCore package Microsoft.EntityFrameworkCore.InMemory --version 9.0.0
   dotnet add SampleWebApiAspNetCore package Swashbuckle.AspNetCore --version 7.2.0
   dotnet add SampleWebApiAspNetCore package System.Linq.Dynamic.Core --version 1.4.10
   ```

### Step 5: Fix Breaking Changes
1. Update namespace imports in code files (Program.cs, Controllers, Extensions):
   - Replace `Microsoft.AspNetCore.Mvc.Versioning` with `Asp.Versioning`
   - Replace `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer` with `Asp.Versioning.ApiExplorer`

2. Review AutoMapper registration in `Program.cs`:
   - Since `AutoMapper.Extensions.Microsoft.DependencyInjection` is removed, verify that AutoMapper 13.x includes DI support natively
   - Expected: `builder.Services.AddAutoMapper(typeof(FoodMappings));` should continue to work

3. Attempt build: `dotnet build --configuration Release`

4. Address any compilation errors:
   - Review compiler messages
   - Consult .NET 9 breaking changes documentation for specific issues
   - Make minimal code changes required for compilation

### Step 6: Runtime Validation
1. Clean and rebuild:
   ```bash
   dotnet clean
   dotnet build --configuration Release
   ```

2. Run the application:
   ```bash
   dotnet run --project SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj
   ```

3. Basic smoke tests:
   - Verify Swagger UI loads: `http://localhost:29435/swagger`
   - Test GET all foods: `http://localhost:29435/api/v1/foods`
   - Test GET single food: `http://localhost:29435/api/v1/foods/1`
   - Verify API versioning works for v1 and v2 endpoints
   - Test HATEOAS links in responses

### Step 7: CI/CD Validation
1. Commit changes: `git commit -m "Migrate to .NET 9"`
2. Push to trigger GitHub Actions workflow
3. Verify workflow completes successfully

### Step 8: Finalize
1. Review all changes
2. Merge migration branch to main (or create PR)
3. Tag release: `git tag v9.0-migration`

## 5. Risks & Compatibility Notes

### Known Breaking Changes (.NET 6 → .NET 9)

**High Impact**:
- **BinaryFormatter removal**: Not used in this project (low risk)
- **API Versioning package replacement**: Requires namespace updates and potential API changes (medium risk)
- **AutoMapper DI extension removal**: AutoMapper 13+ includes DI support natively, but registration syntax may differ (medium risk)

**Medium Impact**:
- **EF Core 9 changes**: Pending migrations throw at runtime; this project uses in-memory database with seed data (low risk for this project)
- **ASP.NET Core middleware changes**: MapStaticAssets and proxy headers changes unlikely to affect this API-only project (low risk)
- **JSON serialization**: Continuing to use Newtonsoft.Json, so minimal impact (low risk)

**Low Impact**:
- **Trimming/AOT**: Not applicable as this project doesn't use ahead-of-time compilation
- **Reflection changes**: AutoMapper uses reflection; version 13.x is compatible with .NET 9

### Potential Blocking Issues

1. **Third-party package compatibility**:
   - **Risk**: `System.Linq.Dynamic.Core` or `Swashbuckle.AspNetCore` may have .NET 9 compatibility issues
   - **Mitigation**: Versions selected are confirmed compatible; fallback to earlier versions if needed

2. **API Versioning migration**:
   - **Risk**: `Asp.Versioning.Mvc` API may differ from `Microsoft.AspNetCore.Mvc.Versioning`
   - **Mitigation**: Both packages maintain similar API surface; consult migration guide if issues arise

3. **AutoMapper breaking changes**:
   - **Risk**: AutoMapper 13.x may have breaking API changes from 11.x
   - **Mitigation**: Review AutoMapper 12.x and 13.x release notes; minimal changes expected for this simple mapping configuration

4. **Swashbuckle .NET 9 support**:
   - **Risk**: Microsoft removed Swashbuckle from default .NET 9 templates
   - **Mitigation**: Swashbuckle remains compatible as a community package; manually include in project

### Dependency Risks

- **No global.json**: Without SDK pinning, different developer machines may use different .NET 9 patch versions
  - **Mitigation**: Add `global.json` to ensure consistency

- **GitHub Actions**: Using `actions/setup-dotnet@v2` may be outdated
  - **Current**: Consider updating to `@v4` for better .NET 9 support
  - **Out of scope**: Stick with v2 unless it fails to install .NET 9

## 6. Validation

### Build Validation

**Pre-migration baseline**:
```bash
git checkout main
dotnet build --configuration Release
# Expected: Build succeeded
```

**Post-migration validation**:
```bash
git checkout migrate-to-dotnet9
dotnet clean
dotnet build --configuration Release
# Expected: Build succeeded with 0 errors, 0 warnings
```

**Package restore verification**:
```bash
dotnet restore
# Expected: All packages restore successfully for net9.0
```

### Runtime Validation

**Application startup**:
```bash
dotnet run --project SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj
# Expected: Application starts on http://localhost:29435
# Expected: No runtime exceptions in console output
```

**Swagger UI validation**:
- Navigate to `http://localhost:29435/swagger`
- Expected: Swagger UI loads successfully
- Expected: Both v1 and v2 API versions appear in version selector

**API endpoint validation**:

1. **GET /api/v1/foods** (HATEOAS with pagination):
   ```bash
   curl http://localhost:29435/api/v1/foods
   ```
   - Expected: 200 OK with `value` array and `links` array
   - Expected: X-Pagination header present
   - Expected: HATEOAS links include self, first, last

2. **GET /api/v1/foods/{id}**:
   ```bash
   curl http://localhost:29435/api/v1/foods/1
   ```
   - Expected: 200 OK with single food item
   - Expected: HATEOAS links present

3. **POST /api/v1/foods**:
   ```bash
   curl -X POST http://localhost:29435/api/v1/foods \
     -H "Content-Type: application/json" \
     -d '{"name":"Test Food","type":"Snack","calories":100,"created":"2025-11-21T00:00:00Z"}'
   ```
   - Expected: 201 Created with Location header

4. **PATCH /api/v1/foods/{id}** (JSON Patch):
   ```bash
   curl -X PATCH http://localhost:29435/api/v1/foods/1 \
     -H "Content-Type: application/json-patch+json" \
     -d '[{"op":"replace","path":"/name","value":"Updated Name"}]'
   ```
   - Expected: 204 No Content or 200 OK

5. **DELETE /api/v1/foods/{id}**:
   ```bash
   curl -X DELETE http://localhost:29435/api/v1/foods/1
   ```
   - Expected: 204 No Content

6. **API v2 endpoint**:
   ```bash
   curl http://localhost:29435/api/v2/foods
   ```
   - Expected: 200 OK (response format depends on v2 implementation)

**AutoMapper validation**:
- Verify POST/PUT operations work correctly (validates mapping from DTOs to Entities)
- No AutoMapper configuration errors in application logs

**EF Core validation**:
- Verify seed data loads on startup (check logs for seed operation)
- Verify CRUD operations work against in-memory database

### CI/CD Validation

**GitHub Actions workflow**:
1. Push migration branch to GitHub
2. Verify workflow triggers and runs
3. Check workflow logs for:
   - .NET 9 SDK installation succeeds
   - `dotnet build --configuration Release` succeeds
   - No errors or warnings in build output

### Performance Baseline (Optional but Recommended)

While performance optimization is out of scope, establishing a baseline helps detect regressions:

**Before migration** (on .NET 6):
- Note application startup time
- Note first request response time

**After migration** (on .NET 9):
- Compare startup time (expected: similar or faster)
- Compare first request response time (expected: similar or faster)

**Note**: .NET 9 generally provides performance improvements; significant regressions indicate an issue.

## Rollback Strategy

If critical issues are encountered during migration:

### Immediate Rollback
```bash
git checkout main
dotnet build --configuration Release
dotnet run --project SampleWebApiAspNetCore/SampleWebApiAspNetCore.csproj
```

### Rollback After Merge
If issues are discovered after merging to main:

1. **Revert commit**:
   ```bash
   git revert <migration-commit-hash>
   git push
   ```

2. **Restore from tag** (if tagged):
   ```bash
   git checkout tags/<pre-migration-tag>
   git checkout -b rollback-branch
   ```

3. **CI/CD rollback**:
   - GitHub Actions will automatically build reverted code
   - No additional CI/CD changes needed

### Partial Rollback
If only specific packages cause issues:

1. Revert problematic package to .NET 6 version temporarily
2. Keep `<TargetFramework>net9.0</TargetFramework>` (most .NET 6 packages work on .NET 9)
3. Investigate and resolve compatibility issue
4. Re-apply package update once resolved

### Data Considerations
- **No data migration required**: Project uses in-memory database with seed data
- **No schema changes**: EF Core migrations not applicable
- **No persistent state**: Rollback has zero data impact

---

## Summary

This migration plan provides a **strict framework version upgrade from .NET 6 to .NET 9** with minimal code changes. The primary changes are:

1. Target framework update: `net6.0` → `net9.0`
2. Package updates to .NET 9-compatible versions
3. API versioning package replacement: `Microsoft.AspNetCore.Mvc.Versioning` → `Asp.Versioning.Mvc`
4. AutoMapper DI extension removal (functionality now built-in)
5. GitHub Actions workflow SDK update: `6.0.x` → `9.0.x`

The migration is low-risk due to:
- Single project with clear dependencies
- In-memory database (no data migration)
- Well-maintained packages with .NET 9 support
- No advanced features (AOT, trimming) that increase complexity

**Estimated effort**: 2-4 hours for migration + validation
**Recommended approach**: Execute all steps in sequence, validate thoroughly at each step

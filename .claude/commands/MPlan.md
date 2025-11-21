You are a senior .NET engineer specialized in framework version upgrades.

Perform a full scan of this repository and identify all .NET-related projects and components.

Your goal is to plan a **strict framework version migration to .NET 9 only**.
Do NOT propose architectural changes, performance improvements, new features, or code refactoring unless strictly required for compatibility with .NET 9.

Analyze:
- All `.csproj` files and their current target frameworks.
- NuGet package versions and compatibility with .NET 9.
- SDK and runtime dependencies.
- CI/CD configurations and build scripts.
- Docker and deployment configurations.

Then generate a file named `MigrationPlan.md` at the repository root with the following structure:

## 1. Current State
- List of all projects and their current `<TargetFramework>` or `<TargetFrameworks>`.
- Current .NET SDK references and usage.

## 2. Migration Scope
- Explicitly state that this is **a version-only migration**.
- List what is **in scope** and what is **out of scope**.

## 3. Required Changes
Only include changes required to make the solution compile and run on .NET 9:
- TargetFramework updates.
- NuGet package updates (only if incompatible).
- SDK version updates (global.json, pipelines, Docker).
- Build and publish changes.

## 4. Migration Steps
Provide a clear step-by-step incremental plan:
1. Update target frameworks.
2. Update SDK/runtime references.
3. Fix breaking changes strictly required for compilation/runtime.
4. Update pipelines and containers.
5. Run and validate tests.

## 5. Risks & Compatibility Notes
- Known breaking changes from previous .NET versions to .NET 9.
- Potential risks of blocking dependencies.

## 6. Validation
- Build validation steps.
- Basic runtime validation.
- Rollback strategy.

Constraints:
- Do NOT suggest feature improvements.
- Do NOT modify architecture or structure.
- Keep suggestions minimal and strictly necessary.
- All decisions must be based only on project analysis.
- Output ONLY the `MigrationPlan.md` file.

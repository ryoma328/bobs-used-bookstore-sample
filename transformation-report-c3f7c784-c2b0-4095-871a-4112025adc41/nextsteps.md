# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build output contains no errors across all five projects:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the transformation to cross-platform .NET has completed without introducing any build-level failures. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run a clean restore and build from the solution root to confirm the error-free state is reproducible in your local environment:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate deprecated APIs or compatibility issues that may surface at runtime.

---

## 2. Run the Unit Tests

Execute the test project to verify that domain logic behaves as expected after the transformation:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failed or skipped tests
- Exceptions that may indicate runtime incompatibilities not caught at build time
- Any tests that were previously passing in the legacy project but are now failing

---

## 3. Verify Data Layer Functionality

The `Bookstore.Data` project likely contains database access logic (e.g., Entity Framework Core migrations or a data context). Perform the following checks:

- Confirm the correct database provider package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql`, or `Sqlite`).
- If using Entity Framework Core, verify migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data
dotnet ef database update --project app/Bookstore.Data
```

- If the legacy project used `System.Data` or another data access approach, manually test database connectivity in your target environment.

---

## 4. Run and Validate the Web Application

Start the `Bookstore.Web` project locally and perform functional validation:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes and pages load correctly.
- Any static assets, configuration files (`appsettings.json`), and middleware are functioning as expected.
- Authentication or authorization mechanisms, if present, work correctly.
- Review any legacy `web.config` settings to confirm they have been migrated to `appsettings.json` or `Program.cs` configuration.

---

## 5. Review Configuration and Environment Settings

Cross-platform .NET applications rely on `appsettings.json` and environment variables rather than `web.config`. Confirm the following:

- Connection strings are correctly defined in `appsettings.json` or environment variables.
- Any environment-specific settings (e.g., `appsettings.Production.json`) are in place.
- Secrets are not hardcoded; consider using the .NET Secret Manager for local development:

```bash
dotnet user-secrets init --project app/Bookstore.Web
dotnet user-secrets set "ConnectionStrings:Default" "your_connection_string"
```

---

## 6. Validate the CDK Project

The `Bookstore.Cdk` project appears to define infrastructure. Verify the following:

- All NuGet dependencies (e.g., `Amazon.CDK` or equivalent) are correctly restored.
- The CDK app synthesizes without errors:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

- Review any hardcoded region, account, or resource names that may need to be updated for your target environment.

---

## 7. Check for Runtime Compatibility Issues

Even with a clean build, certain legacy APIs may have behavioral differences on cross-platform .NET. Review the following areas:

- **File paths**: Ensure no hardcoded Windows-style paths (backslashes) exist. Use `Path.Combine()` throughout.
- **Encoding**: Confirm `Encoding.Default` usage has been replaced with explicit encodings such as `Encoding.UTF8`.
- **Registry or Windows-specific APIs**: Confirm none are referenced, as these will fail on Linux/macOS.
- **`System.Drawing`**: If used, replace with a cross-platform alternative such as `SkiaSharp` or `ImageSharp`.

---

## 8. Publish the Application

Once validation is complete, publish the web application for deployment:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Verify the contents of the `./publish` directory to ensure all required files, assets, and configuration are present before deploying to your target environment.
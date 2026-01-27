---
name: migrating-dotnet
description: Use when upgrading .NET Framework (3.5, 4.x) to modern .NET (6, 8, 9, 10), migrating between .NET versions, encountering compatibility issues during upgrades, or planning .NET modernization projects
---

# Migrating .NET Versions

## Overview

.NET migration transforms applications from legacy .NET Framework to modern .NET (or between modern versions). With **GitHub Copilot app modernization** (VS 2022 17.14.16+ / VS 2026), direct jumps from .NET Framework 3.5/4.8 to .NET 10 are now feasible.

**Core principle:** Prepare meticulously, let AI automate transformations, validate obsessively.

**Expected gains:** 30-50% performance improvement, 40-60% memory reduction, modern security features.

## When to Use

**Use when:**
- Upgrading from .NET Framework 3.5/4.x to .NET 6/8/9/10
- Moving between modern .NET versions (e.g., .NET 6 → .NET 8)
- End-of-support deadlines approaching
- Need cross-platform deployment, containerization, or cloud-native features
- Performance improvements required (30-50% faster HTTP, 40-60% lower memory)

**Don't use for:**
- Minor patch updates within same version
- Projects that will be retired soon

## Migration Decision Flow

```
Is app simple with few dependencies?
├─ YES → In-place migration (update TFM directly)
└─ NO → Incremental migration (Strangler Fig pattern with YARP)
    │
    └─ Start with leaf dependencies, work up to entry point
```

## Prerequisites (Complete Before Migration)

| Step | Action | Why |
|------|--------|-----|
| 1 | Upgrade to .NET Framework 4.7.2+ | Latest APIs compatible with modern .NET |
| 2 | Convert to SDK-style .csproj | Required for modern .NET and tooling |
| 3 | Migrate packages.config → PackageReference | Modern dependency management |
| 4 | Implement Central Package Management | Single source of truth for versions |
| 5 | Replace Newtonsoft.Json → System.Text.Json | Faster, more secure, modern standard |
| 6 | Move config to appsettings.json | Structured configuration model |
| 7 | Decouple business logic into .NET Standard 2.0 libraries | Clean isolation boundary |
| 8 | Ensure all tests pass (green baseline) | Safety net for validation |

### Central Package Management (CPM)

Create `Directory.Packages.props` at solution root:
```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Microsoft.Extensions.Logging" Version="8.0.0" />
    <PackageVersion Include="System.Text.Json" Version="8.0.0" />
  </ItemGroup>
</Project>
```

## Quick Reference: Breaking Changes by Area

| Category | Removed/Changed | Replacement |
|----------|-----------------|-------------|
| **AppDomains** | Removed | AssemblyLoadContext or separate processes |
| **Remoting** | Removed | gRPC, REST APIs, or named pipes |
| **WCF Server** | Removed | CoreWCF (community), gRPC, or REST |
| **Web Forms** | Removed | Blazor, Razor Pages, or MVC |
| **System.Web** | Removed | ASP.NET Core middleware |
| **app.config/web.config** | Changed | appsettings.json + IConfiguration |
| **Global.asax** | Removed | Program.cs + middleware pipeline |
| **HTTP Modules/Handlers** | Removed | Middleware |
| **Windows Auth** | Changed | Azure AD or platform-specific auth |

## Migration Strategy

### Phase 1: Shared Libraries (2-4 weeks typical)

1. Target .NET Standard 2.0 for dual compatibility
2. Multi-target to support both old and new hosts:
   ```xml
   <TargetFrameworks>net48;netstandard2.0</TargetFrameworks>
   ```
3. Add platform guards for Windows-specific code:
   ```csharp
   if (OperatingSystem.IsWindows())
   {
       // Windows-specific code
   }
   ```

### Phase 2: Data Access Layer

- **EF6 → EF Core:** Different query behavior, especially lazy loading and raw SQL
- Test all queries—don't assume equivalent behavior
- Consider database migration for licensing cost reduction

### Phase 3: Business Logic

- Usually migrates cleanly
- Watch for: Registry access, Windows paths, COM interop

### Phase 4: Application Entry Point

For ASP.NET → ASP.NET Core:

1. Create new ASP.NET Core project
2. Set up YARP reverse proxy to route between old/new
3. Migrate routes incrementally
4. Remove old routes as new ones are validated

## Tools

### GitHub Copilot App Modernization (Primary Tool)

**Requirements:**
- Visual Studio 2022 (17.14.16+) or Visual Studio 2026
- GitHub Copilot subscription (Pro, Pro+, Business, or Enterprise)
- .NET 10 SDK installed

**Replaces these extensions (uninstall if present):**
- .NET Upgrade Assistant
- GitHub Copilot App Modernization — Upgrade for .NET
- Azure Migrate Application and Code Assessment for .NET

**Invoke:**
1. Right-click solution → "Modernize", or
2. Copilot Chat (Agent Mode): `@modernize upgrade my entire solution to .NET 10 LTS`

**Key capabilities:**
- Agent-based planning (understands project dependencies)
- Automated code transformations
- Learns from manual interventions
- Git integration with automatic commits at each phase

### Other Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **upgrade-assistant CLI** | Analysis and reports | If not using VS with Copilot |
| **API Portability Analyzer** | Check API compatibility | Early assessment |
| **try-convert** | Convert .csproj to SDK-style | Pre-migration prep |

```bash
dotnet tool install -g upgrade-assistant
upgrade-assistant analyze <project-path>
```

## Third-Party Dependencies

1. **Check NuGet for .NET 6+ versions** of each package
2. **No compatible version?** Options:
   - Find alternative package
   - Fork and port (if open source)
   - Wrap in .NET Standard 2.0 shim
   - Replace functionality
3. **Multi-target during transition** to maintain compatibility

## Testing Strategy

| Test Type | When | Focus |
|-----------|------|-------|
| **Unit tests** | Each component migration | Logic correctness |
| **Integration tests** | After layer migration | Cross-component behavior |
| **Performance tests** | Before/after comparison | No regressions |
| **End-to-end tests** | Full path validation | User flows work |

**Critical:** Run existing test suite against migrated code—tests passing immediately indicate coverage gaps.

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| **Sync-over-Async (deadlocks)** | Never use `.Result` or `.Wait()`. Use `await` throughout. |
| **EF Core "Could Not Be Translated"** | Call `ToList()` before client-side filtering (watch for N+1) |
| **BinaryFormatter usage** | Removed for security. Migrate to `System.Text.Json` |
| **Big-bang migration** | Use incremental approach for complex apps |
| **Ignoring behavioral changes** | Test thoroughly; some APIs behave differently |
| **Not updating CI/CD** | Update build pipelines to new SDK |
| **Skipping performance baseline** | Benchmark before migration to detect regressions |
| **Secrets in logs/errors** | Use structured logging with data masking |
| **Binding redirect errors** | Use CPM (`Directory.Packages.props`) to unify versions |

### Sync-over-Async Example
```csharp
// ❌ WRONG - causes deadlocks
var result = MyAsyncMethod().Result;

// ✅ CORRECT
var result = await MyAsyncMethod();
```

## Version-Specific Notes

### .NET 8 (LTS - Current)
- Native AOT compilation available
- Minimal APIs improvements
- Support until November 2026

### .NET 9 (STS)
- System.Linq.AsyncEnumerable in core
- OpenAPI 3.1 support
- Short-term support

### .NET 10 (LTS - November 2025)
- W3C trace context default
- Generic math shift behavior changes
- `--interactive` defaults to true in CLI

## Zero Trust Security (Post-Migration)

Implement during or immediately after migration:

### Managed Identity + Azure Key Vault
```csharp
var builder = WebApplication.CreateBuilder(args);

// Use Managed Identity to authenticate with Azure Key Vault
var keyVaultUrl = new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/");
builder.Configuration.AddAzureKeyVault(keyVaultUrl, new DefaultAzureCredential());

// Connection strings retrieved from Key Vault, not hard-coded
builder.Services.AddDbContext<MyContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Database Access with Managed Identity
```sql
-- In Azure SQL Database
CREATE USER [your-app-name] FROM EXTERNAL PROVIDER;
ALTER ROLE db_owner ADD MEMBER [your-app-name];
```

### Security Checklist
- [ ] No hard-coded secrets in code or config
- [ ] All secrets in Azure Key Vault via Managed Identity
- [ ] Database uses Managed Identity (no SQL passwords)
- [ ] Microsoft Entra ID authentication (not local stores)
- [ ] HTTPS enforced; TLS 1.3 default
- [ ] CORS policies explicitly configured (no wildcards)
- [ ] API endpoints require valid bearer tokens
- [ ] Secrets never logged or in error messages

## Technology Blockers & Solutions

### ASP.NET Web Forms (.aspx) - No Automatic Migration

| Option | Effort | Best For |
|--------|--------|----------|
| **Blazor Server** | Medium | Forms-heavy apps, minimal logic rewrite |
| **Blazor WebAssembly** | Medium-High | Full client-side SPA |
| **Razor Pages** | High | Simpler page-focused development |
| **ASP.NET Core MVC** | High | Full control, complex apps |

### WCF Server-Side - No Automatic Migration

| Option | Effort | Best For |
|--------|--------|----------|
| **CoreWCF** | Low | Quick win, existing WCF services as-is |
| **gRPC** | Medium | Modern, high-performance, long-term |
| **ASP.NET Core Web API** | High | REST redesign, maximum flexibility |

### System.Web Dependencies

| Old | New |
|-----|-----|
| `HttpContext.Current` | `IHttpContextAccessor` (DI) |
| `Session["key"]` | `ISession` with `IDistributedCache` |
| `Server.MapPath()` | `Path.Combine(env.ContentRootPath, relativePath)` |
| `Application` cache | `IMemoryCache` or `IDistributedCache` |

## Red Flags - STOP and Reassess

- WCF server-side code (significant rewrite needed)
- Heavy System.Web dependencies (middleware rewrite)
- COM+ usage (limited support)
- SQL CLR stored procedures (may need redesign)
- Custom MSBuild targets (review compatibility)
- Application Domains / .NET Remoting (architecture redesign needed)

## Migration Checklist

### Pre-Migration
- [ ] All projects converted to SDK-style format
- [ ] Central Package Management (`Directory.Packages.props`) implemented
- [ ] Business logic decoupled into separate libraries
- [ ] System.Text.Json adoption complete
- [ ] Configuration moved to appsettings.json model
- [ ] Complex dependencies identified and isolated
- [ ] All NuGet packages audited for compatibility
- [ ] Existing tests pass (green baseline)
- [ ] Secrets removed from config files and code
- [ ] Azure Key Vault access confirmed
- [ ] Dedicated migration branch created

### Execution
- [ ] Invoke GitHub Copilot app modernization
- [ ] Review generated plan before proceeding
- [ ] Monitor and intervene when needed
- [ ] Migrate shared libraries → data layer → business logic → entry point
- [ ] Implement Zero Trust security (Managed Identity, Key Vault)
- [ ] Update CI/CD pipelines

### Post-Migration Validation
- [ ] Application starts without errors
- [ ] All unit/integration tests passing
- [ ] No hard-coded secrets in code or config
- [ ] All secrets via Azure Key Vault
- [ ] Database authenticated via Managed Identity
- [ ] Performance meets or exceeds baseline
- [ ] Security scan passed (no new vulnerabilities)
- [ ] Load testing completed (10x expected traffic)
- [ ] EF Core queries return correct results
- [ ] Rollback procedure documented and tested

### Deployment
- [ ] Blue-green or canary deployment configured
- [ ] Application Insights monitoring enabled
- [ ] Alerts configured (error rate, latency, Key Vault access)
- [ ] On-call rotation prepared
- [ ] Runbook created for common issues

## Estimated Timeline

| Size | Projects | Duration |
|------|----------|----------|
| Small | 5-10 | 2-4 weeks |
| Medium | 10-30 | 2-3 months |
| Large | 30+ | 6-12 months |

*GitHub Copilot accelerates by 30-50% compared to manual upgrades.*

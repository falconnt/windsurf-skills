---
name: migrating-dotnet
description: Use when upgrading .NET Framework (3.5, 4.x) to modern .NET (6, 8, 9, 10), migrating between .NET versions, encountering compatibility issues during upgrades, or planning .NET modernization projects
---

# Migrating .NET Versions

## Overview

.NET migration transforms applications from legacy .NET Framework to modern .NET (or between modern versions). Unlike simple updates, this requires addressing architectural differences, breaking changes, and dependency compatibility.

**Core principle:** Migrate incrementally with validation at each step—never big-bang entire codebases.

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
| 2 | Migrate packages.config → PackageReference | Modern dependency management |
| 3 | Convert to SDK-style .csproj | Required for modern .NET |
| 4 | Run .NET Upgrade Assistant analysis | Identify scope and breaking changes |

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

| Tool | Purpose | Limitations |
|------|---------|-------------|
| **.NET Upgrade Assistant** | Automated analysis, project conversion | Won't fix WCF/Remoting/AppDomains |
| **API Portability Analyzer** | Check API compatibility | May miss runtime behavioral changes |
| **try-convert** | Convert .csproj to SDK-style | Manual review still needed |
| **GitHub Copilot upgrade4.net** | AI-assisted upgrade with agent mode | Requires VS/VS Code with Copilot |

**Run analysis first:**
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

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Big-bang migration | Use incremental approach for complex apps |
| Ignoring behavioral changes | Test thoroughly; some APIs behave differently |
| Not updating CI/CD | Update build pipelines to new SDK |
| Skipping performance baseline | Benchmark before migration to detect regressions |
| Assuming NuGet packages "just work" | Verify each dependency has compatible version |
| Forgetting configuration migration | appsettings.json structure differs from web.config |

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

## Red Flags - STOP and Reassess

- WCF server-side code (significant rewrite needed)
- Heavy System.Web dependencies (middleware rewrite)
- COM+ usage (limited support)
- SQL CLR stored procedures (may need redesign)
- Custom MSBuild targets (review compatibility)

## Migration Checklist

```markdown
## Assessment
- [ ] Run .NET Upgrade Assistant analysis
- [ ] Document all third-party dependencies and compatibility
- [ ] Identify System.Web usage
- [ ] Estimate effort per component
- [ ] Define target .NET version (prefer LTS: 8 or 10)

## Prerequisites
- [ ] Upgrade to .NET Framework 4.7.2+
- [ ] Convert packages.config → PackageReference
- [ ] Convert to SDK-style .csproj
- [ ] Set up parallel test environment

## Execution
- [ ] Migrate shared libraries to .NET Standard 2.0
- [ ] Migrate data access layer
- [ ] Migrate business logic
- [ ] Migrate entry point (web app, console, etc.)
- [ ] Update CI/CD pipelines

## Validation
- [ ] All tests passing
- [ ] Performance baseline met or exceeded
- [ ] Security scan clean
- [ ] Production monitoring configured
```

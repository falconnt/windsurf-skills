---
name: using-logic-apps-consumption
description: Use when building, designing, or coding Azure Logic Apps Consumption workflows - covers architecture patterns, Code View JSON conventions, error handling, performance optimization, security, naming conventions, and deployment best practices
---

# Azure Logic Apps Consumption Best Practices

## Overview

Azure Logic Apps Consumption is a multitenant, pay-per-execution integration platform. Success requires **loosely coupled architecture**, **idempotent design**, and **defensive error handling**. This skill covers Code View (JSON) conventions and production-ready patterns.

## When to Use

- Building new Logic Apps Consumption workflows
- Reviewing or refactoring existing workflows
- Debugging performance or reliability issues
- Planning CI/CD deployment pipelines

**Not for:** Logic Apps Standard (single-tenant) - different patterns apply.

## Architecture Patterns

### Loosely Coupled Design

```
Source System → Service Bus → Logic App (Enrich/Route) → Service Bus → Logic App (Transform) → Destination
```

**Benefits:**
- **Parallel execution** - independent components scale separately
- **Isolated failures** - one component failure doesn't cascade
- **Independent deployment** - update modules without full redeploy
- **Focused testing** - unit test each workflow separately

**Rule:** One workflow per integration concern. Avoid monolithic workflows handling multiple protocols or systems.

### Idempotent Operations

Same message processed multiple times must yield identical results:
- Use **Upsert** (update or insert) instead of always INSERT
- Generate **deterministic IDs** that remain consistent across retries
- Implement **message deduplication** at Service Bus level

## Code View (JSON) Best Practices

### Schema Reference

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0"
}
```

### Structure Order

1. Parameters (environment-specific values)
2. Triggers
3. Actions
4. Outputs

### Parameters for Environment Abstraction

```json
"parameters": {
  "environment": { "type": "String", "defaultValue": "dev" },
  "apiEndpoint": { "type": "String" },
  "connectionString": { "type": "SecureString" }
}
```

**Use `SecureString`/`SecureObject`** for credentials - masked in logs and history.

### Meaningful Action Names

```json
// ❌ BAD
"HTTP": { ... },
"Compose_2": { ... },
"For_each_3": { ... }

// ✅ GOOD
"Get_Customer_Details": { ... },
"Transform_To_Canonical_Format": { ... },
"Process_Each_Order_Line": { ... }
```

### Run-After Configuration

```json
"Send_Error_Notification": {
  "runAfter": {
    "Process_Order": ["Failed", "TimedOut"]
  }
}
```

**Statuses:** `Succeeded`, `Failed`, `Skipped`, `TimedOut`

## Error Handling

### Retry Policies

```json
"retryPolicy": {
  "type": "Exponential",
  "count": 4,
  "interval": "PT7S",
  "minimumInterval": "PT5S",
  "maximumInterval": "PT1H"
}
```

| Policy | Use Case |
|--------|----------|
| `Default` | Most scenarios (4 retries, exponential) |
| `None` | Non-retryable operations |
| `Fixed` | Predictable timing needed |
| `Exponential` | Transient failures (408, 429, 5xx) |

### Scope-Based Error Handling

```
┌─ Scope: Process_Order ─────────────────┐
│  Get_Order → Validate → Transform      │
└────────────────────────────────────────┘
         │
         ├── [Succeeded] → Send_Confirmation
         └── [Failed] → Log_Error → Send_Alert
```

Group related actions in **Scope**, then use `runAfter` on scope status.

### Terminate Action

```json
"Terminate_With_Error": {
  "type": "Terminate",
  "inputs": {
    "runStatus": "Failed",
    "runError": {
      "code": "ValidationFailed",
      "message": "@{variables('errorMessage')}"
    }
  }
}
```

## Performance Optimization

### Limits (Consumption)

| Resource | Limit |
|----------|-------|
| Actions per 5 min | 100,000 (300,000 high throughput) |
| Concurrent outbound calls | 2,500 |
| HTTP request size | 100 MB |
| Run duration | 90 days |
| For Each iterations | 100,000 |

### Optimization Techniques

1. **Avoid nested loops** - each iteration persists state to storage
2. **Filter before looping** - reduce iterations with `@take()`, `@skip()`, conditions
3. **Use chunking** for large messages (>100MB)
4. **Debatch at trigger** - split array into individual runs
5. **Limit concurrency** on For Each (default: 20, max: 50)

```json
"For_each": {
  "type": "Foreach",
  "runtimeConfiguration": {
    "concurrency": { "repetitions": 10 }
  }
}
```

### Trigger Optimization

```json
"triggers": {
  "When_messages_available": {
    "recurrence": {
      "frequency": "Minute",
      "interval": 3
    },
    "splitOn": "@triggerBody()",
    "runtimeConfiguration": {
      "concurrency": { "runs": 25 }
    }
  }
}
```

**`splitOn`** debatches arrays into parallel runs.

## Security Best Practices

### Authentication

1. **Managed Identity** (preferred) - no credentials to manage
2. **Azure Key Vault** for secrets - reference via `@parameters('keyVaultSecret')`
3. **OAuth 2.0** for external APIs

### Network Security

- **Private endpoints** for internal resources
- **IP restrictions** on HTTP triggers
- **Access keys** for trigger URLs (regenerate periodically)

### Data Protection

- Use **Secure Inputs/Outputs** on sensitive actions (masks in run history)
- Avoid logging PII in tracked properties
- Enable **customer-managed keys** for encryption at rest

```json
"Get_Secret": {
  "type": "ApiConnection",
  "inputs": { ... },
  "runtimeConfiguration": {
    "secureData": {
      "properties": ["inputs", "outputs"]
    }
  }
}
```

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Logic App | `la-{system}-{function}-{env}` | `la-orders-processing-prod` |
| Actions | PascalCase with underscores | `Get_Customer_By_Id` |
| Variables | camelCase | `orderTotal`, `isValid` |
| Parameters | camelCase | `apiEndpoint`, `maxRetries` |
| Connections | `conn-{service}-{purpose}` | `conn-sql-orders` |

## Expressions & Functions

### Common Patterns

```javascript
// Null-safe property access
@coalesce(triggerBody()?['customer']?['name'], 'Unknown')

// Date formatting
@formatDateTime(utcNow(), 'yyyy-MM-dd')

// Conditional
@if(equals(variables('status'), 'active'), 'Process', 'Skip')

// Array operations
@length(body('Get_Items'))
@first(body('Get_Items'))
@union(variables('list1'), variables('list2'))

// String operations
@concat('Order-', variables('orderId'))
@replace(variables('text'), ' ', '-')
@split(variables('csv'), ',')
```

### Expression Limits

- Max expression length: 8,192 characters
- Max nesting depth: 10 levels
- Avoid complex expressions in loops (evaluate once, store in variable)

## Deployment (ARM Templates)

### Template Structure

```
├── azuredeploy.json          # Main template
├── azuredeploy.parameters.dev.json
├── azuredeploy.parameters.prod.json
└── connectors/
    └── connections.json      # Shared connections
```

### CI/CD Pipeline

```yaml
# Azure DevOps example
stages:
  - stage: Deploy
    jobs:
      - job: DeployLogicApp
        steps:
          - task: AzureResourceManagerTemplateDeployment@3
            inputs:
              deploymentScope: 'Resource Group'
              templateLocation: 'Linked artifact'
              csmFile: 'azuredeploy.json'
              csmParametersFile: 'azuredeploy.parameters.$(env).json'
```

### Connection Authorization

After deployment, connections require **manual authorization** or use:
- Managed Identity (auto-authorized)
- Service Principal with pre-configured consent

## Monitoring & Diagnostics

### Enable Diagnostic Settings

```json
"diagnosticSettings": {
  "logs": [
    { "category": "WorkflowRuntime", "enabled": true },
    { "category": "IntegrationAccountTrackingEvents", "enabled": true }
  ],
  "metrics": [
    { "category": "AllMetrics", "enabled": true }
  ]
}
```

### Key Metrics

- **Runs Started/Succeeded/Failed**
- **Run Latency**
- **Action Latency**
- **Billable Action Executions**

### Tracked Properties

```json
"Send_Order": {
  "trackedProperties": {
    "orderId": "@variables('orderId')",
    "customerName": "@body('Get_Customer')?['name']"
  }
}
```

Query in Log Analytics:
```kusto
AzureDiagnostics
| where ResourceType == "WORKFLOWS"
| where trackedProperties_orderId_s == "12345"
```

## Common Pitfalls

### SetVariable Cannot Self-Reference

```json
// ❌ FAILS - variable cannot reference itself
"Set_Counter": {
  "type": "SetVariable",
  "inputs": {
    "name": "counter",
    "value": "@add(variables('counter'), 1)"
  }
}

// ✅ WORKS - use intermediate variable or IncrementVariable
"Increment_Counter": {
  "type": "IncrementVariable",
  "inputs": {
    "name": "counter",
    "value": 1
  }
}
```

**For non-numeric accumulation (strings, arrays):**
```json
// ❌ FAILS - cannot append to itself
"Append_To_List": {
  "type": "SetVariable",
  "inputs": {
    "name": "items",
    "value": "@union(variables('items'), array(body('Get_Item')))"
  }
}

// ✅ WORKS - use AppendToArrayVariable
"Append_Item": {
  "type": "AppendToArrayVariable",
  "inputs": {
    "name": "items",
    "value": "@body('Get_Item')"
  }
}
```

### Variables Inside Parallel Branches

Variables set in parallel branches have **race conditions** - final value is unpredictable.

```
// ❌ DANGEROUS
┌─ Parallel ─────────────────────┐
│  Branch A: Set_Total = 100     │
│  Branch B: Set_Total = 200     │
└────────────────────────────────┘
// Total could be 100 OR 200
```

**Solution:** Use separate variables per branch, merge after parallel completes.

### Compose vs Variable

| Use Case | Compose | Variable |
|----------|---------|----------|
| Single use, immediate | ✅ | ❌ overkill |
| Reuse across actions | ❌ | ✅ |
| Inside loops (accumulate) | ❌ | ✅ |
| Transform data once | ✅ | ❌ |

`Compose` outputs are immutable and cheaper (no storage write).

### Trigger Outputs in Loops

`triggerBody()` and `triggerOutputs()` are **evaluated once** at workflow start. Safe to use in loops without re-fetching.

### Until Loop Gotchas

```json
"Until_Complete": {
  "type": "Until",
  "expression": "@equals(variables('status'), 'Complete')",
  "limit": {
    "count": 60,
    "timeout": "PT1H"
  }
}
```

**Pitfalls:**
- Default limit is 60 iterations AND 1 hour - whichever hits first
- No built-in delay between iterations - add explicit `Delay` action
- Infinite loop protection exists but burns through action quota fast

### HTTP Action Timeout

Default HTTP timeout is **100 seconds**. Long-running APIs need async pattern:

```json
"Call_Long_Running_API": {
  "type": "Http",
  "inputs": {
    "method": "POST",
    "uri": "...",
    "retryPolicy": { "type": "None" }
  },
  "operationOptions": "DisableAsyncPattern"
}
```

For truly long operations, use **webhook pattern** or **polling with Until**.

### JSON Parse Failures

`Parse JSON` action fails silently on schema mismatch if properties are missing. Use:

```json
// Make properties optional in schema
"properties": {
  "name": { "type": "string" },
  "email": { "type": ["string", "null"] }
}
```

Or use `coalesce()` after parsing:
```javascript
@coalesce(body('Parse_JSON')?['email'], 'no-email@example.com')
```

### Split On with Empty Arrays

`splitOn` on empty array = **zero workflow runs** (no error, no notification).

Add validation or use Service Bus dead-letter for empty payload scenarios.

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Monolithic workflow | Hard to debug, deploy, scale | Split into focused workflows |
| Hardcoded values | Can't promote between environments | Use parameters |
| No error handling | Silent failures | Scope + runAfter + alerts |
| Nested For Each | Exponential storage writes | Flatten or use child workflows |
| Polling when push available | Unnecessary executions | Use webhooks/Service Bus |
| Large inline data | Bloats definition | Store in Blob, reference |
| Synchronous long operations | Timeouts | Use async pattern with callbacks |

## Cost Optimization

### Billing Model

- ~4,000 free actions/month
- Standard connectors: lower rate
- Enterprise connectors: higher rate
- Each action execution counts

### Reduction Strategies

1. **Trigger conditions** - prevent unnecessary runs
2. **Batch operations** - 1 bulk call vs N individual calls
3. **Remove unused connectors** - eliminate authorization overhead
4. **Audit regularly** - disable/delete unused workflows

```json
"triggers": {
  "When_email_arrives": {
    "conditions": [
      { "expression": "@contains(triggerBody()?['from'], '@important.com')" }
    ]
  }
}
```

## Quick Reference

| Task | Approach |
|------|----------|
| Handle transient errors | Retry policy (Exponential) |
| Handle permanent errors | Scope + runAfter[Failed] + Terminate |
| Process large arrays | splitOn at trigger OR chunked For Each |
| Store secrets | Key Vault + Managed Identity |
| Debug failures | Run history + tracked properties + Log Analytics |
| Deploy safely | ARM templates + parameter files + CI/CD |
| Optimize cost | Trigger conditions + batch operations + audit |

## References

- [Workflow Definition Language Schema](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-definition-language)
- [Error Handling](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-exception-handling)
- [Limits and Configuration](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config)
- [ARM Template Deployment](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-azure-resource-manager-templates-overview)

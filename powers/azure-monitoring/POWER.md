---
name: "azure-monitoring"
displayName: "Azure Monitoring"
description: "Query logs with KQL, analyze metrics, and troubleshoot Azure resources - observability and diagnostics with read-only access"
keywords: ["monitor", "logs", "kql", "kusto", "metrics", "alerts", "app-insights", "grafana", "troubleshoot", "health"]
author: "Volodymyr Marynychev"
---

# Azure Monitoring Power

## Quick Start

```bash
# 1. Verify prerequisites
node --version        # Need v20+
az --version          # Need Azure CLI installed

# 2. Authenticate with Azure
az login

# 3. Verify authentication
az account show
```

After installing, try:
- *"List my Azure subscriptions"*
- *"Check resource health events in my subscription"*
- *"Query activity logs for resource X"*

## Overview

Monitor, analyze, and troubleshoot Azure resources with comprehensive observability tools.

**Key capabilities:**
- **Log Analytics**: Query logs using KQL across workspaces and resources
- **Metrics**: Query and analyze resource metrics
- **Activity Logs**: Track control plane operations
- **Resource Health**: Check availability status and service health events
- **Application Insights**: Access code optimization recommendations
- **Workbooks**: Manage interactive data visualization dashboards
- **Grafana**: List Azure Managed Grafana workspaces
- **AppLens**: AI-powered diagnostics and troubleshooting

**Authentication**: Requires `az login` with read-only permissions.

## Available MCP Tools

### Scope Navigation
- `subscription_list` - List all Azure subscriptions
- `group_list` - List resource groups in a subscription

### monitor (Log Analytics, Metrics, Activity Logs)
Use `learn=true` to discover subcommands:
- `monitor_workspace_list` - List Log Analytics workspaces
- `monitor_workspace_log_query` - Query logs across entire workspace (KQL)
- `monitor_resource_log_query` - Query logs for specific resource (KQL)
- `monitor_table_list` - List tables in a workspace
- `monitor_table_type_list` - List table types
- `monitor_metrics_query` - Query metrics for a resource
- `monitor_metrics_definitions` - List available metrics for a resource
- `monitor_activitylog_list` - Query activity logs
- `monitor_healthmodels_entity_get` - Get health model entity status
- `monitor_webtests_list/get/create/update` - Manage web tests

### resourcehealth (Resource Availability)
- `resourcehealth_availability-status_get` - Get health status for specific resource
- `resourcehealth_availability-status_list` - List health status for all resources
- `resourcehealth_health-events_list` - List service health events

### applicationinsights
- `applicationinsights_recommendation_list` - List code optimization recommendations

### grafana
- `grafana_list` - List Azure Managed Grafana workspaces

### workbooks
- `workbooks_list` - List workbooks in a resource group
- `workbooks_show` - Get workbook details
- `workbooks_create/update/delete` - Manage workbooks

### applens (AI Diagnostics)
- `applens_resource_diagnose` - AI-powered diagnostics for Azure resources

> **Tip:** Use `learn=true` parameter on hierarchical tools to discover available subcommands and their parameters.

## MCP Server Configuration

```json
{
  "mcpServers": {
    "azure-mcp": {
      "command": "npx",
      "args": [
        "-y", "@azure/mcp@latest", "server", "start",
        "--namespace", "monitor",
        "--namespace", "resourcehealth", 
        "--namespace", "applicationinsights",
        "--namespace", "workbooks",
        "--namespace", "grafana",
        "--namespace", "applens"
      ],
      "env": {
        "AZURE_MCP_COLLECT_TELEMETRY": "false"
      }
    }
  }
}
```

### Optional: Auto-Approve Tools

To skip approval prompts for read-only monitoring operations, manually add `autoApprove` to your global config at `~/.kiro/settings/mcp.json`:

```json
"power-azure-monitoring-azure-mcp": {
  ...
  "autoApprove": [
    "subscription_list",
    "group_list",
    "monitor",
    "resourcehealth",
    "applicationinsights",
    "grafana",
    "applens",
    "workbooks"
  ]
}
```

> **Note:** autoApprove must be added manually after power installation - it's not copied from the power config for security reasons.

## Available Steering Files

- **kql-patterns.md** - Common KQL queries, performance optimization
- **monitoring-workflows.md** - Troubleshooting workflows, alerting best practices

## Workflow Examples

### Check Resource Health
```javascript
// Step 1: List subscriptions
subscription_list({})

// Step 2: List health events
resourcehealth({ 
  intent: "List health events",
  command: "resourcehealth_health-events_list",
  parameters: { subscription: "your-sub-id" }
})
```

### Query Logs with KQL
```javascript
// Step 1: List workspaces
monitor({
  intent: "List workspaces",
  command: "monitor_workspace_list",
  parameters: { subscription: "your-sub-id" }
})

// Step 2: Query logs
monitor({
  intent: "Query error logs",
  command: "monitor_workspace_log_query",
  parameters: {
    subscription: "your-sub-id",
    "resource-group": "your-rg",
    workspace: "your-workspace",
    table: "AzureDiagnostics",
    query: "errors",
    hours: 24
  }
})
```

### Diagnose Resource Issues
```javascript
applens({
  intent: "Diagnose app service issues",
  command: "applens_resource_diagnose",
  parameters: {
    subscription: "your-sub-id",
    "resource-group": "your-rg",
    resource: "your-app-service",
    "resource-type": "Microsoft.Web/sites",
    question: "Why is my app slow?"
  }
})
```

## Common KQL Patterns

### Error Analysis
```kql
AzureDiagnostics
| where Level == "Error"
| summarize count() by bin(TimeGenerated, 1h), Category
| render timechart
```

### Performance Percentiles
```kql
AppServiceHTTPLogs
| summarize 
    p50 = percentile(TimeTaken, 50),
    p95 = percentile(TimeTaken, 95),
    p99 = percentile(TimeTaken, 99)
  by bin(TimeGenerated, 15m)
```

### Dependency Failures
```kql
AppDependencies
| where Success == false
| summarize count() by Target, DependencyType, ResultCode
| order by count_ desc
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Authentication required` | Run `az login` |
| `Only 2 tools visible` | Remove `--namespace` flag from mcp.json args |
| `Resource not found` | Verify resource exists and you have access |
| `Tools not loading` | Ensure global `~/.kiro/settings/mcp.json` doesn't override power config |

## Related Powers

| Power | Use Case |
|-------|----------|
| [azure-architect](../azure-architect/) | Design monitoring architecture (no auth required) |
| [azure-operations](../azure-operations/) | Manage resources, configure diagnostics |

## Resources

- [Azure MCP Server](https://github.com/microsoft/mcp/tree/main/servers/Azure.Mcp.Server)
- [Azure MCP Documentation](https://learn.microsoft.com/azure/developer/azure-mcp-server/)
- [KQL Quick Reference](https://docs.microsoft.com/azure/data-explorer/kql-quick-reference)
- [Azure Monitor Documentation](https://docs.microsoft.com/azure/azure-monitor/)

---

**Package:** `@azure/mcp` | **License:** MIT

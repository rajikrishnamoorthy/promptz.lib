# Monitoring Workflows Guide

This steering file provides guidance for KQL patterns, troubleshooting workflows, and alerting best practices using the azure-monitoring power.

## Troubleshooting Decision Tree

Use this decision tree to guide your troubleshooting approach:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Troubleshooting Workflow                     │
└─────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
              ┌─────────────────────────────────────────┐
              │           What's the symptom?           │
              └─────────────────────────────────────────┘
                     │              │              │
               Application      Performance     Resource
                 Errors          Issues         Unavailable
                     │              │              │
                     ▼              ▼              ▼
         ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
         │ 1. Check health │ │ 1. Query metrics│ │ 1. Check health │
         │ 2. Query errors │ │ 2. Find slow    │ │ 2. Service      │
         │ 3. Check deps   │ │    requests     │ │    events       │
         │ 4. Activity log │ │ 3. Check deps   │ │ 3. Activity log │
         └─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Quick Reference: Common Scenarios

| Scenario | First Tool | Follow-up |
|----------|------------|-----------|
| App returning 5xx errors | `monitor_resource_log_query` | Check `AppExceptions` table |
| Slow response times | `monitor_metrics_query` | Query `AppServiceHTTPLogs` for slow requests |
| Resource unavailable | `resourcehealth_availability-status_get` | Check `resourcehealth_health-events_list` |
| Deployment broke something | `monitor_activitylog_list` | Filter by `status: Failed` |
| Cross-service issues | `monitor_workspace_log_query` | Use `union` to correlate logs |

## Troubleshooting Workflows

### Workflow 1: Application Error Investigation

When users report errors or you see elevated error rates:

```javascript
// Step 1: Check resource health first - is Azure having issues?
resourcehealth_availability-status_get({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}"
})

// Step 2: Query for HTTP 5xx errors with timeline
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppServiceHTTPLogs
    | where ScStatus >= 500
    | summarize ErrorCount = count() by ScStatus, bin(TimeGenerated, 15m)
    | order by TimeGenerated desc
  `,
  "timespan": "PT6H"
})

// Step 3: Get exception details
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppExceptions
    | project TimeGenerated, ExceptionType, OuterMessage, InnermostMessage, OperationId
    | order by TimeGenerated desc
    | take 50
  `,
  "timespan": "PT6H"
})

// Step 4: Check if dependencies are failing
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppDependencies
    | where Success == false
    | summarize FailureCount = count() by Target, DependencyType, ResultCode
    | order by FailureCount desc
  `,
  "timespan": "PT6H"
})

// Step 5: Check for recent changes that might have caused the issue
monitor_activitylog_list({
  "resource-group": "{rg}",
  "timespan": "P1D"
})
```

### Workflow 2: Performance Degradation Investigation

When response times are slow or users report performance issues:

```javascript
// Step 1: Query CPU and memory metrics
monitor_metrics_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "metric-names": "CpuPercentage,MemoryPercentage,RequestsInApplicationQueue",
  "timespan": "PT6H",
  "interval": "PT5M",
  "aggregation": "Average"
})

// Step 2: Calculate response time percentiles
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppServiceHTTPLogs
    | summarize 
        p50 = percentile(TimeTaken, 50),
        p95 = percentile(TimeTaken, 95),
        p99 = percentile(TimeTaken, 99),
        RequestCount = count()
      by bin(TimeGenerated, 15m)
    | order by TimeGenerated desc
  `,
  "timespan": "PT6H"
})

// Step 3: Find the slowest endpoints
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppServiceHTTPLogs
    | summarize 
        AvgDuration = avg(TimeTaken),
        P95Duration = percentile(TimeTaken, 95),
        RequestCount = count()
      by CsUriStem
    | where RequestCount > 10
    | order by P95Duration desc
    | take 20
  `,
  "timespan": "PT6H"
})

// Step 4: Check dependency performance
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppDependencies
    | summarize 
        AvgDuration = avg(DurationMs),
        P95Duration = percentile(DurationMs, 95),
        CallCount = count(),
        FailureRate = countif(Success == false) * 100.0 / count()
      by Target, DependencyType
    | order by P95Duration desc
  `,
  "timespan": "PT6H"
})
```

### Workflow 3: Resource Availability Investigation

When a resource appears unavailable or unhealthy:

```javascript
// Step 1: Check current availability status
resourcehealth_availability-status_get({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}"
})

// Step 2: Check for Azure service health events
resourcehealth_health-events_list({
  "subscription": "{sub}",
  "event-type": "ServiceIssue"
})

// Step 3: Check for planned maintenance
resourcehealth_health-events_list({
  "subscription": "{sub}",
  "event-type": "PlannedMaintenance"
})

// Step 4: Review activity logs for failed operations
monitor_activitylog_list({
  "resource-group": "{rg}",
  "timespan": "P1D",
  "status": "Failed"
})

// Step 5: Check for configuration changes
monitor_activitylog_list({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "timespan": "P1D"
})
```

### Workflow 4: Cross-Service Correlation

When issues span multiple services:

```javascript
// Step 1: List available Log Analytics workspaces
monitor_workspace_list({
  "resource-group": "monitoring-rg"
})

// Step 2: Query across all services for errors
monitor_workspace_log_query({
  "workspace-id": "{workspace-id}",
  "query": `
    union AppExceptions, AppDependencies, AppRequests
    | where TimeGenerated > ago(1h)
    | where Success == false or isnotempty(ExceptionType)
    | summarize ErrorCount = count() by Type, AppRoleName
    | order by ErrorCount desc
  `,
  "timespan": "PT1H"
})

// Step 3: Trace a specific operation across services
monitor_workspace_log_query({
  "workspace-id": "{workspace-id}",
  "query": `
    union AppRequests, AppDependencies, AppTraces, AppExceptions
    | where OperationId == "{operation-id}"
    | project TimeGenerated, Type, Name, Success, DurationMs, Message, ExceptionType
    | order by TimeGenerated asc
  `,
  "timespan": "PT1H"
})

// Step 4: Find correlated failures
monitor_workspace_log_query({
  "workspace-id": "{workspace-id}",
  "query": `
    AppRequests
    | where Success == false
    | join kind=inner (
        AppExceptions
      ) on OperationId
    | project TimeGenerated, RequestName = Name, ExceptionType, OuterMessage
    | take 50
  `,
  "timespan": "PT1H"
})
```

### Workflow 5: Deployment Impact Analysis

After a deployment, verify everything is working:

```javascript
// Step 1: Check activity logs for deployment events
monitor_activitylog_list({
  "resource-group": "{rg}",
  "timespan": "PT2H"
})

// Step 2: Compare error rates before and after deployment
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppServiceHTTPLogs
    | where TimeGenerated > ago(4h)
    | extend Period = iff(TimeGenerated > ago(2h), "After", "Before")
    | summarize 
        TotalRequests = count(),
        ErrorCount = countif(ScStatus >= 500),
        ErrorRate = round(countif(ScStatus >= 500) * 100.0 / count(), 2)
      by Period
  `,
  "timespan": "PT4H"
})

// Step 3: Check for new exception types
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppExceptions
    | where TimeGenerated > ago(2h)
    | summarize Count = count() by ExceptionType
    | order by Count desc
  `,
  "timespan": "PT2H"
})

// Step 4: Verify response times haven't degraded
monitor_resource_log_query({
  "resource-id": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{app}",
  "query": `
    AppServiceHTTPLogs
    | where TimeGenerated > ago(4h)
    | extend Period = iff(TimeGenerated > ago(2h), "After", "Before")
    | summarize 
        AvgResponseTime = avg(TimeTaken),
        P95ResponseTime = percentile(TimeTaken, 95)
      by Period
  `,
  "timespan": "PT4H"
})
```

## Alerting Best Practices

### Alert Design Principles

1. **Alert on symptoms, not causes** - Alert when users are impacted, not on every metric spike
2. **Use appropriate thresholds** - Base thresholds on historical data, not guesses
3. **Implement severity levels** - Critical for immediate action, Warning for investigation
4. **Avoid alert fatigue** - Too many alerts leads to ignored alerts
5. **Include runbooks** - Every alert should link to troubleshooting steps

### Recommended Alert Categories

| Category | What to Monitor | Severity |
|----------|-----------------|----------|
| Availability | Resource health, endpoint availability | Critical |
| Errors | HTTP 5xx rate, exception rate | Critical/Warning |
| Performance | Response time P95, CPU/Memory | Warning |
| Saturation | Queue depth, connection pool | Warning |
| Business | Failed transactions, SLA breaches | Critical |

### Alert Threshold Guidelines

```
┌─────────────────────────────────────────────────────────────────┐
│                    Alert Threshold Strategy                     │
└─────────────────────────────────────────────────────────────────┘

Error Rate Alerts:
├── Critical: > 5% of requests failing (5xx)
├── Warning:  > 1% of requests failing (5xx)
└── Info:     > 0.1% of requests failing (5xx)

Response Time Alerts:
├── Critical: P95 > 5x baseline
├── Warning:  P95 > 2x baseline
└── Info:     P95 > 1.5x baseline

Resource Utilization:
├── Critical: CPU/Memory > 90% for 10 minutes
├── Warning:  CPU/Memory > 75% for 15 minutes
└── Info:     CPU/Memory > 60% for 30 minutes

Availability:
├── Critical: Resource unavailable for > 1 minute
└── Warning:  Resource degraded for > 5 minutes
```

### KQL Queries for Alert Rules

#### Error Rate Alert
```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(5m)
| summarize 
    TotalRequests = count(),
    ErrorCount = countif(ScStatus >= 500)
| extend ErrorRate = ErrorCount * 100.0 / TotalRequests
| where ErrorRate > 5
```

#### Response Time Alert
```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(5m)
| summarize P95ResponseTime = percentile(TimeTaken, 95)
| where P95ResponseTime > 3000  // 3 seconds
```

#### Dependency Failure Alert
```kql
AppDependencies
| where TimeGenerated > ago(5m)
| summarize 
    TotalCalls = count(),
    FailedCalls = countif(Success == false)
| extend FailureRate = FailedCalls * 100.0 / TotalCalls
| where FailureRate > 10
```

## Monitoring Setup Checklist

### Initial Setup
- [ ] Log Analytics workspace created
- [ ] Diagnostic settings enabled on all resources
- [ ] Application Insights connected to applications
- [ ] Resource health alerts configured
- [ ] Activity log alerts for critical operations

### Application Monitoring
- [ ] Request logging enabled
- [ ] Exception tracking configured
- [ ] Dependency tracking enabled
- [ ] Custom metrics defined
- [ ] Availability tests created

### Infrastructure Monitoring
- [ ] Platform metrics collected
- [ ] Resource health monitoring enabled
- [ ] Activity log forwarding configured
- [ ] Network watcher enabled
- [ ] Security center connected

### Alerting
- [ ] Critical alerts defined and tested
- [ ] Alert action groups configured
- [ ] Escalation procedures documented
- [ ] Runbooks linked to alerts
- [ ] Alert suppression rules for maintenance

## Time Range Reference

| Timespan | Format | Use Case |
|----------|--------|----------|
| 1 hour | `PT1H` | Real-time troubleshooting |
| 6 hours | `PT6H` | Recent issue investigation |
| 24 hours | `P1D` | Daily patterns |
| 7 days | `P7D` | Weekly trends |
| 30 days | `P30D` | Monthly analysis |

## Metric Aggregation Reference

| Aggregation | Use Case |
|-------------|----------|
| `Average` | General resource utilization |
| `Maximum` | Peak detection, capacity planning |
| `Minimum` | Baseline identification |
| `Total` | Request counts, data transfer |
| `Count` | Event frequency |

## Common Troubleshooting Patterns

### Pattern: Intermittent Errors

```javascript
// Look for patterns in error timing
monitor_resource_log_query({
  "resource-id": "{resource-id}",
  "query": `
    AppServiceHTTPLogs
    | where ScStatus >= 500
    | summarize ErrorCount = count() by bin(TimeGenerated, 5m)
    | render timechart
  `,
  "timespan": "P1D"
})
```

### Pattern: Memory Leak Detection

```javascript
// Track memory growth over time
monitor_metrics_query({
  "resource-id": "{resource-id}",
  "metric-names": "MemoryWorkingSet",
  "timespan": "P1D",
  "interval": "PT1H",
  "aggregation": "Average"
})
```

### Pattern: Connection Pool Exhaustion

```javascript
// Check for connection-related errors
monitor_resource_log_query({
  "resource-id": "{resource-id}",
  "query": `
    AppDependencies
    | where DependencyType == "SQL" or DependencyType == "HTTP"
    | where Success == false
    | where ResultCode contains "timeout" or ResultCode contains "connection"
    | summarize Count = count() by Target, ResultCode, bin(TimeGenerated, 15m)
  `,
  "timespan": "PT6H"
})
```

## Related Steering Files

- **[kql-patterns.md](./kql-patterns.md)** - Common KQL queries, performance optimization, query templates

## Resources

- [Azure Monitor Documentation](https://docs.microsoft.com/azure/azure-monitor/)
- [KQL Quick Reference](https://docs.microsoft.com/azure/data-explorer/kql-quick-reference)
- [Azure Monitor Alerts](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-overview)
- [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)
- [Log Analytics Tutorial](https://docs.microsoft.com/azure/azure-monitor/logs/log-analytics-tutorial)

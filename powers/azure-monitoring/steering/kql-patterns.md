# KQL Patterns Guide

This steering file provides common KQL (Kusto Query Language) queries, performance optimization techniques, and query templates for the azure-monitoring power.

## KQL Quick Reference

### Query Structure

```kql
TableName                           // Start with a table
| where TimeGenerated > ago(1h)     // Filter rows
| project Column1, Column2          // Select columns
| summarize count() by Column1      // Aggregate
| order by count_ desc              // Sort
| take 100                          // Limit results
```

### Essential Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `where Status == "Error"` |
| `project` | Select columns | `project Name, Value` |
| `extend` | Add calculated columns | `extend Duration = EndTime - StartTime` |
| `summarize` | Aggregate data | `summarize count() by Category` |
| `order by` | Sort results | `order by TimeGenerated desc` |
| `take` | Limit rows | `take 100` |
| `join` | Combine tables | `join kind=inner Table2 on Key` |
| `union` | Combine tables vertically | `union Table1, Table2` |
| `render` | Visualize results | `render timechart` |

## Common Query Patterns

### Error Analysis Patterns

#### Count Errors by Type
```kql
// Count errors grouped by status code over time
AppServiceHTTPLogs
| where TimeGenerated > ago(24h)
| where ScStatus >= 400
| summarize Count = count() by ScStatus, bin(TimeGenerated, 1h)
| order by TimeGenerated desc
```

#### Error Rate Calculation
```kql
// Calculate error rate as percentage
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| summarize 
    TotalRequests = count(),
    ErrorCount = countif(ScStatus >= 500),
    ErrorRate = round(countif(ScStatus >= 500) * 100.0 / count(), 2)
```

#### Top Errors by Endpoint
```kql
// Find which endpoints have the most errors
AppServiceHTTPLogs
| where TimeGenerated > ago(24h)
| where ScStatus >= 500
| summarize ErrorCount = count() by CsUriStem, ScStatus
| order by ErrorCount desc
| take 20
```

#### Exception Analysis
```kql
// Analyze exceptions with full details
AppExceptions
| where TimeGenerated > ago(24h)
| summarize 
    Count = count(),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
  by ExceptionType, OuterMessage
| order by Count desc
| take 20
```

#### Error Trend Detection
```kql
// Detect if errors are increasing
AppServiceHTTPLogs
| where TimeGenerated > ago(2h)
| summarize ErrorCount = countif(ScStatus >= 500) by bin(TimeGenerated, 10m)
| order by TimeGenerated asc
| extend PrevErrorCount = prev(ErrorCount)
| extend Trend = iff(ErrorCount > PrevErrorCount * 1.5, "Increasing", "Stable")
```

### Performance Analysis Patterns

#### Response Time Percentiles
```kql
// Calculate response time percentiles
AppServiceHTTPLogs
| where TimeGenerated > ago(6h)
| summarize 
    p50 = percentile(TimeTaken, 50),
    p90 = percentile(TimeTaken, 90),
    p95 = percentile(TimeTaken, 95),
    p99 = percentile(TimeTaken, 99),
    avg = avg(TimeTaken)
  by bin(TimeGenerated, 15m)
| order by TimeGenerated desc
```

#### Slow Requests
```kql
// Find requests slower than threshold
AppServiceHTTPLogs
| where TimeGenerated > ago(6h)
| where TimeTaken > 3000  // 3 seconds
| project TimeGenerated, CsUriStem, CsMethod, TimeTaken, ScStatus
| order by TimeTaken desc
| take 50
```

#### Endpoint Performance Comparison
```kql
// Compare performance across endpoints
AppServiceHTTPLogs
| where TimeGenerated > ago(24h)
| summarize 
    RequestCount = count(),
    AvgDuration = avg(TimeTaken),
    P95Duration = percentile(TimeTaken, 95),
    ErrorRate = round(countif(ScStatus >= 500) * 100.0 / count(), 2)
  by CsUriStem
| where RequestCount > 100
| order by P95Duration desc
| take 20
```

#### Performance Degradation Detection
```kql
// Compare current hour to previous day same hour
let currentHour = AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| summarize CurrentP95 = percentile(TimeTaken, 95);
let previousDay = AppServiceHTTPLogs
| where TimeGenerated between (ago(25h) .. ago(24h))
| summarize PreviousP95 = percentile(TimeTaken, 95);
currentHour
| extend PreviousP95 = toscalar(previousDay)
| extend DegradationPercent = round((CurrentP95 - PreviousP95) * 100.0 / PreviousP95, 2)
```

### Dependency Analysis Patterns

#### Dependency Performance
```kql
// Analyze dependency call performance
AppDependencies
| where TimeGenerated > ago(6h)
| summarize 
    CallCount = count(),
    AvgDuration = avg(DurationMs),
    P95Duration = percentile(DurationMs, 95),
    FailureRate = round(countif(Success == false) * 100.0 / count(), 2)
  by Target, DependencyType
| order by P95Duration desc
```

#### Failing Dependencies
```kql
// Find failing external dependencies
AppDependencies
| where TimeGenerated > ago(1h)
| where Success == false
| summarize 
    FailureCount = count(),
    LastFailure = max(TimeGenerated)
  by Target, DependencyType, ResultCode
| order by FailureCount desc
```

#### Dependency Chain Analysis
```kql
// Trace dependency calls for slow requests
AppRequests
| where TimeGenerated > ago(1h)
| where DurationMs > 5000
| join kind=inner (
    AppDependencies
    | where TimeGenerated > ago(1h)
  ) on OperationId
| project 
    RequestTime = TimeGenerated,
    RequestName = Name,
    RequestDuration = DurationMs,
    DependencyTarget = Target,
    DependencyDuration = DurationMs1
| order by RequestDuration desc
| take 50
```

### Resource Utilization Patterns

#### CPU and Memory Trends
```kql
// Track resource utilization over time
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 1h), Computer
| render timechart
```

#### Resource Saturation Detection
```kql
// Find periods of high resource usage
Perf
| where TimeGenerated > ago(24h)
| where (ObjectName == "Processor" and CounterName == "% Processor Time")
    or (ObjectName == "Memory" and CounterName == "% Committed Bytes In Use")
| summarize AvgValue = avg(CounterValue) by ObjectName, bin(TimeGenerated, 15m)
| where AvgValue > 80
| order by TimeGenerated desc
```

### Activity Log Patterns

#### Failed Operations
```kql
// Find failed Azure operations
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue == "Failed"
| project TimeGenerated, OperationNameValue, ResourceGroup, Caller, ActivityStatusValue
| order by TimeGenerated desc
```

#### Configuration Changes
```kql
// Track configuration changes
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue contains "write" or OperationNameValue contains "delete"
| summarize ChangeCount = count() by OperationNameValue, ResourceGroup, bin(TimeGenerated, 1d)
| order by TimeGenerated desc
```

#### Operations by User
```kql
// See who made changes
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "write" or OperationNameValue contains "delete"
| summarize OperationCount = count() by Caller, OperationNameValue
| order by OperationCount desc
```

### Cross-Service Correlation Patterns

#### Trace Request Flow
```kql
// Follow a request across services
union AppRequests, AppDependencies, AppTraces, AppExceptions
| where OperationId == "{operation-id}"
| project 
    TimeGenerated,
    Type,
    Name = coalesce(Name, Message, ExceptionType),
    Duration = coalesce(DurationMs, 0),
    Success = coalesce(Success, true),
    Details = coalesce(OuterMessage, ResultCode, "")
| order by TimeGenerated asc
```

#### Service-to-Service Errors
```kql
// Find errors that propagate across services
AppRequests
| where TimeGenerated > ago(1h)
| where Success == false
| join kind=inner (
    AppDependencies
    | where Success == false
  ) on OperationId
| summarize 
    Count = count()
  by RequestName = Name, DependencyTarget = Target, DependencyResultCode = ResultCode
| order by Count desc
```

#### Multi-Service Health Overview
```kql
// Get health summary across all services
union AppRequests, AppDependencies
| where TimeGenerated > ago(1h)
| summarize 
    TotalCalls = count(),
    FailedCalls = countif(Success == false),
    AvgDuration = avg(DurationMs),
    P95Duration = percentile(DurationMs, 95)
  by Type, AppRoleName
| extend FailureRate = round(FailedCalls * 100.0 / TotalCalls, 2)
| order by FailureRate desc
```

## Query Optimization Techniques

### Performance Best Practices

1. **Filter early** - Apply `where` clauses as early as possible
2. **Use time filters** - Always include `TimeGenerated` filter
3. **Limit columns** - Use `project` to select only needed columns
4. **Avoid wildcards** - Use specific column names instead of `*`
5. **Use summarize wisely** - Aggregate before joining

### Optimized vs Unoptimized Examples

#### ❌ Unoptimized
```kql
// Bad: Scans all data, then filters
AppServiceHTTPLogs
| project *
| where TimeGenerated > ago(1h)
| where ScStatus >= 500
```

#### ✅ Optimized
```kql
// Good: Filters first, selects specific columns
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| where ScStatus >= 500
| project TimeGenerated, CsUriStem, ScStatus, TimeTaken
```

#### ❌ Unoptimized Join
```kql
// Bad: Joins before filtering
AppRequests
| join AppDependencies on OperationId
| where TimeGenerated > ago(1h)
| where Success == false
```

#### ✅ Optimized Join
```kql
// Good: Filters before joining
AppRequests
| where TimeGenerated > ago(1h)
| where Success == false
| join kind=inner (
    AppDependencies
    | where TimeGenerated > ago(1h)
  ) on OperationId
```

### Query Size Management

```kql
// Use take to limit results during development
AppServiceHTTPLogs
| where TimeGenerated > ago(24h)
| take 1000  // Limit for testing

// Use sample for large datasets
AppServiceHTTPLogs
| where TimeGenerated > ago(7d)
| sample 10000  // Random sample
```

## Template Queries

### Template: Service Health Dashboard

```kql
// Comprehensive service health view
let timeRange = 1h;
let errorThreshold = 1;
AppServiceHTTPLogs
| where TimeGenerated > ago(timeRange)
| summarize 
    TotalRequests = count(),
    SuccessfulRequests = countif(ScStatus < 400),
    ClientErrors = countif(ScStatus >= 400 and ScStatus < 500),
    ServerErrors = countif(ScStatus >= 500),
    AvgResponseTime = avg(TimeTaken),
    P95ResponseTime = percentile(TimeTaken, 95)
| extend 
    SuccessRate = round(SuccessfulRequests * 100.0 / TotalRequests, 2),
    ErrorRate = round(ServerErrors * 100.0 / TotalRequests, 2)
| extend 
    HealthStatus = case(
        ErrorRate > 5, "Critical",
        ErrorRate > errorThreshold, "Warning",
        "Healthy"
    )
```

### Template: Anomaly Detection

```kql
// Detect anomalies in request patterns
let baselineStart = ago(7d);
let baselineEnd = ago(1d);
let currentStart = ago(1h);
let baseline = AppServiceHTTPLogs
| where TimeGenerated between (baselineStart .. baselineEnd)
| summarize 
    BaselineAvg = avg(TimeTaken),
    BaselineStdDev = stdev(TimeTaken);
AppServiceHTTPLogs
| where TimeGenerated > currentStart
| extend 
    BaselineAvg = toscalar(baseline | project BaselineAvg),
    BaselineStdDev = toscalar(baseline | project BaselineStdDev)
| extend ZScore = (TimeTaken - BaselineAvg) / BaselineStdDev
| where abs(ZScore) > 3  // More than 3 standard deviations
| project TimeGenerated, CsUriStem, TimeTaken, ZScore
| order by abs(ZScore) desc
```

### Template: SLA Compliance Report

```kql
// Calculate SLA compliance
let slaTarget = 99.9;  // 99.9% availability
let responseTimeTarget = 2000;  // 2 seconds
AppServiceHTTPLogs
| where TimeGenerated > ago(30d)
| summarize 
    TotalRequests = count(),
    SuccessfulRequests = countif(ScStatus < 500),
    FastRequests = countif(TimeTaken < responseTimeTarget)
  by bin(TimeGenerated, 1d)
| extend 
    AvailabilityPercent = round(SuccessfulRequests * 100.0 / TotalRequests, 3),
    PerformancePercent = round(FastRequests * 100.0 / TotalRequests, 3)
| extend 
    AvailabilitySLAMet = AvailabilityPercent >= slaTarget,
    PerformanceSLAMet = PerformancePercent >= slaTarget
| order by TimeGenerated desc
```

### Template: User Impact Analysis

```kql
// Estimate user impact from errors
AppRequests
| where TimeGenerated > ago(1h)
| where Success == false
| summarize 
    AffectedOperations = dcount(OperationId),
    AffectedUsers = dcount(UserId),
    TotalFailures = count()
  by Name
| order by AffectedUsers desc
```

## Table Reference

### Application Insights Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `AppRequests` | Incoming requests | Name, DurationMs, Success, ResultCode |
| `AppDependencies` | Outgoing calls | Target, DependencyType, DurationMs, Success |
| `AppExceptions` | Exceptions | ExceptionType, OuterMessage, OperationId |
| `AppTraces` | Log messages | Message, SeverityLevel |
| `AppMetrics` | Custom metrics | Name, Sum, Count |
| `AppPageViews` | Page views | Name, DurationMs, Url |

### Azure Diagnostics Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `AzureDiagnostics` | Resource logs | ResourceType, Category, OperationName |
| `AzureActivity` | Activity logs | OperationNameValue, ActivityStatusValue, Caller |
| `AzureMetrics` | Platform metrics | MetricName, Average, Maximum |

### App Service Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `AppServiceHTTPLogs` | HTTP request logs | CsUriStem, ScStatus, TimeTaken |
| `AppServiceConsoleLogs` | Console output | ResultDescription |
| `AppServiceAppLogs` | Application logs | Level, Message |

## Time Functions Reference

| Function | Example | Result |
|----------|---------|--------|
| `ago(1h)` | `where TimeGenerated > ago(1h)` | Last hour |
| `now()` | `extend CurrentTime = now()` | Current time |
| `startofday()` | `startofday(TimeGenerated)` | Start of day |
| `endofday()` | `endofday(TimeGenerated)` | End of day |
| `bin()` | `bin(TimeGenerated, 1h)` | Round to hour |
| `datetime_diff()` | `datetime_diff('minute', end, start)` | Time difference |

## Aggregation Functions Reference

| Function | Description | Example |
|----------|-------------|---------|
| `count()` | Count rows | `summarize count()` |
| `countif()` | Conditional count | `countif(Status == "Error")` |
| `dcount()` | Distinct count | `dcount(UserId)` |
| `sum()` | Sum values | `sum(BytesSent)` |
| `avg()` | Average | `avg(DurationMs)` |
| `min()` / `max()` | Min/Max | `min(TimeGenerated)` |
| `percentile()` | Percentile | `percentile(DurationMs, 95)` |
| `stdev()` | Standard deviation | `stdev(DurationMs)` |

## Related Steering Files

- **[monitoring-workflows.md](./monitoring-workflows.md)** - Troubleshooting workflows, alerting best practices

## Resources

- [KQL Quick Reference](https://docs.microsoft.com/azure/data-explorer/kql-quick-reference)
- [KQL Tutorial](https://docs.microsoft.com/azure/data-explorer/kusto/query/tutorial)
- [Log Analytics Tables](https://docs.microsoft.com/azure/azure-monitor/reference/tables/tables-category)
- [Application Insights Data Model](https://docs.microsoft.com/azure/azure-monitor/app/data-model)
- [Query Best Practices](https://docs.microsoft.com/azure/data-explorer/kusto/query/best-practices)

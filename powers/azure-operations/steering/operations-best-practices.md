# Operations Best Practices Guide

This steering file provides guidance for Azure resource management patterns, naming conventions, and operational excellence using the azure-operations power.

## Resource Management Decision Tree

```
   ┌─────────────────────────────────────────────────────────────────┐
   │              Resource Management Strategy                       │
   └─────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
              ┌─────────────────────────────────────────┐
              │           What are you managing?        │
              └─────────────────────────────────────────┘
                     │              │              │
                  Storage       Databases      Containers
                     │              │              │
                     ▼              ▼              ▼
         ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
         │ storage_*       │ │ cosmos_*, sql_* │ │ aks_*, acr_*    │
         │ keyvault_*      │ │ postgres_*      │ │ functionapp_*   │
         │                 │ │ mysql_*, redis_*│ │ appservice_*    │
         └─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Naming Conventions

### Azure Resource Naming Pattern

Follow this pattern for consistent resource naming:

```
{resource-type-abbreviation}-{workload}-{environment}-{region}-{instance}
```

> **Note:** Storage Accounts and Container Registries don't support hyphens, so use condensed lowercase alphanumeric format: `{abbreviation}{workload}{environment}{region}{instance}`

**Examples:**
- `stmyappprodeus001` (Storage Account - no hyphens)
- `kv-myapp-prod-eus-001` (Key Vault)
- `cosmos-myapp-prod-eus-001` (Cosmos DB)
- `aks-myapp-prod-eus-001` (AKS Cluster)

### Resource Type Abbreviations

| Resource Type | Abbreviation | Example |
|---------------|--------------|---------|
| Storage Account | `st` | `stmyappprodeus001` |
| Key Vault | `kv` | `kv-myapp-prod-eus-001` |
| Cosmos DB | `cosmos` | `cosmos-myapp-prod-eus` |
| SQL Server | `sql` | `sql-myapp-prod-eus` |
| SQL Database | `sqldb` | `sqldb-myapp-prod-eus` |
| PostgreSQL | `psql` | `psql-myapp-prod-eus` |
| MySQL | `mysql` | `mysql-myapp-prod-eus` |
| Redis Cache | `redis` | `redis-myapp-prod-eus` |
| AKS Cluster | `aks` | `aks-myapp-prod-eus` |
| Container Registry | `acr` | `acrmyappprodeus001` |
| Function App | `func` | `func-myapp-prod-eus` |
| App Service | `app` | `app-myapp-prod-eus` |
| Event Hub Namespace | `evhns` | `evhns-myapp-prod` |
| Service Bus Namespace | `sbns` | `sbns-myapp-prod` |
| Event Grid Topic | `evgt` | `evgt-myapp-prod` |
| SignalR | `sigr` | `sigr-myapp-prod` |
| Resource Group | `rg` | `rg-myapp-prod-eus` |

### Environment Abbreviations

| Environment | Abbreviation |
|-------------|--------------|
| Development | `dev` |
| Testing | `test` |
| Staging | `stg` |
| Production | `prod` |
| Shared | `shared` |

### Region Abbreviations

| Region | Abbreviation |
|--------|--------------|
| East US | `eus` |
| East US 2 | `eus2` |
| West US | `wus` |
| West US 2 | `wus2` |
| Central US | `cus` |
| North Europe | `neu` |
| West Europe | `weu` |
| UK South | `uks` |
| Southeast Asia | `sea` |

## Resource Organization Patterns

### Resource Group Strategy

Organize resources by workload and environment:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Subscription                                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │ rg-myapp-dev    │  │ rg-myapp-stg    │  │ rg-myapp-prod   │  │
│  │                 │  │                 │  │                 │  │
│  │ • App Service   │  │ • App Service   │  │ • App Service   │  │
│  │ • SQL Database  │  │ • SQL Database  │  │ • SQL Database  │  │
│  │ • Key Vault     │  │ • Key Vault     │  │ • Key Vault     │  │
│  │ • Storage       │  │ • Storage       │  │ • Storage       │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ rg-shared-services                                      │    │
│  │ • Container Registry (shared across environments)       │    │
│  │ • Log Analytics Workspace                               │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Listing Resources by Group

```javascript
// List all storage accounts in a specific resource group
storage_account_list({
  "resource-group": "rg-myapp-prod"
})

// List all Key Vaults in a resource group
keyvault_list({
  "resource-group": "rg-myapp-prod"
})

// List all databases in a resource group
cosmos_account_list({
  "resource-group": "rg-myapp-prod"
})
```

## Storage Management Patterns

### Storage Account Best Practices

1. **Use appropriate access tiers** - Hot for frequent access, Cool for infrequent, Archive for long-term
2. **Enable soft delete** - Protect against accidental deletion
3. **Use private endpoints** - Secure access over private network
4. **Enable versioning** - Track changes to blobs
5. **Implement lifecycle policies** - Automate tier transitions and deletion

### Storage Operations Workflow

```javascript
// Step 1: List storage accounts to find the right one
storage_account_list({
  "resource-group": "rg-data-prod"
})

// Step 2: List containers in the storage account
storage_container_list({
  "account-name": "stmyappprod001"
})

// Step 3: List blobs with a specific prefix
storage_blob_list({
  "account-name": "stmyappprod001",
  "container-name": "documents",
  "prefix": "2024/"
})
```

### Storage Organization Pattern

```
Storage Account: stmyappprod001
├── containers/
│   ├── documents/          # User documents
│   │   ├── 2026/
│   │   └── 2025/
│   ├── backups/            # Database backups
│   ├── logs/               # Application logs
│   └── temp/               # Temporary files (with lifecycle policy)
└── tables/
    └── audit-logs          # Audit trail data
```

## Database Management Patterns

### Database Selection Guide

| Use Case | Recommended Service | Tool Prefix |
|----------|---------------------|-------------|
| Document/JSON data | Cosmos DB | `cosmos_*` |
| Relational data (Microsoft stack) | Azure SQL | `sql_*` |
| Relational data (Open source) | PostgreSQL | `postgres_*` |
| Relational data (MySQL compatible) | MySQL | `mysql_*` |
| Caching/Sessions | Redis | `redis_*` |

### Cosmos DB Operations

```javascript
// Step 1: List Cosmos DB accounts
cosmos_account_list({
  "resource-group": "rg-data-prod"
})

// Step 2: List databases in an account
cosmos_database_list({
  "account-name": "cosmos-myapp-prod",
  "resource-group": "rg-data-prod"
})

// Step 3: Query data with pagination
cosmos_query({
  "account-name": "cosmos-myapp-prod",
  "database-name": "orders",
  "container-name": "items",
  "query": "SELECT TOP 100 * FROM c WHERE c.status = 'pending' ORDER BY c._ts DESC"
})
```

### SQL Database Operations

```javascript
// Step 1: List SQL servers
sql_server_list({
  "resource-group": "rg-data-prod"
})

// Step 2: List databases on a server
sql_database_list({
  "server-name": "sql-myapp-prod",
  "resource-group": "rg-data-prod"
})
```

## Container Management Patterns

### AKS Cluster Operations

```javascript
// Step 1: List AKS clusters
aks_cluster_list({
  "resource-group": "rg-k8s-prod"
})

// Step 2: Review node pools for scaling decisions
aks_nodepool_list({
  "cluster-name": "aks-myapp-prod",
  "resource-group": "rg-k8s-prod"
})

// Step 3: List container registries
acr_list({
  "resource-group": "rg-shared-services"
})
```

### Container Registry Best Practices

1. **Use geo-replication** - For multi-region deployments
2. **Enable content trust** - Sign and verify images
3. **Implement retention policies** - Clean up old images
4. **Use private endpoints** - Secure registry access
5. **Scan images for vulnerabilities** - Enable Microsoft Defender

## Secrets Management Patterns

### Key Vault Best Practices

1. **One Key Vault per environment** - Separate dev/staging/prod secrets
2. **Use managed identities** - Avoid storing credentials in code
3. **Enable soft delete** - Protect against accidental deletion
4. **Enable purge protection** - Prevent permanent deletion
5. **Rotate secrets regularly** - Implement secret rotation policies

### Key Vault Operations

```javascript
// Step 1: List Key Vaults
keyvault_list({
  "resource-group": "rg-myapp-prod"
})

// Step 2: List secrets (names only)
keyvault_secret_list({
  "vault-name": "kv-myapp-prod-001"
})

// Step 3: List cryptographic keys
keyvault_key_list({
  "vault-name": "kv-myapp-prod-001"
})
```

### Secrets Organization Pattern

```
Key Vault: kv-myapp-prod-001
├── secrets/
│   ├── database-connection-string
│   ├── storage-account-key
│   ├── api-key-external-service
│   └── certificate-password
├── keys/
│   ├── data-encryption-key
│   └── signing-key
└── certificates/
    ├── ssl-certificate
    └── client-certificate
```

## Messaging Services Patterns

### Event-Driven Architecture Selection

| Pattern | Service | Use Case |
|---------|---------|----------|
| Event streaming | Event Hubs | High-throughput telemetry, logs |
| Message queuing | Service Bus | Reliable message delivery, transactions |
| Event routing | Event Grid | React to Azure resource events |
| Real-time | SignalR | Live updates, notifications |

### Messaging Operations

```javascript
// List Event Hubs namespaces
eventhubs_namespace_list({
  "resource-group": "rg-messaging-prod"
})

// List Service Bus namespaces
servicebus_namespace_list({
  "resource-group": "rg-messaging-prod"
})

// List Event Grid topics
eventgrid_topic_list({
  "resource-group": "rg-messaging-prod"
})

// List SignalR services
signalr_list({
  "resource-group": "rg-messaging-prod"
})
```

## Operational Excellence Checklist

Use this checklist for operational readiness:

### Resource Organization
- [ ] Consistent naming convention applied
- [ ] Resources organized by workload and environment
- [ ] Resource groups properly scoped
- [ ] Tags applied for cost allocation and ownership

### Data Protection
- [ ] Soft delete enabled on storage and Key Vault
- [ ] Backup policies configured for databases
- [ ] Geo-redundancy enabled for critical data
- [ ] Encryption at rest enabled

### Access Control
- [ ] Managed identities used where possible
- [ ] RBAC roles follow least privilege
- [ ] Service principals have minimal permissions
- [ ] Access reviews scheduled

### Monitoring
- [ ] Diagnostic settings configured
- [ ] Alerts set for critical metrics
- [ ] Log Analytics workspace connected
- [ ] Azure Advisor recommendations reviewed

## Resource Inventory Workflow

### Complete Environment Audit

```javascript
// Step 1: List all storage accounts
storage_account_list({
  "subscription": "my-subscription"
})

// Step 2: List all databases
cosmos_account_list({ "subscription": "my-subscription" })
sql_server_list({ "subscription": "my-subscription" })
postgres_server_list({ "subscription": "my-subscription" })
mysql_server_list({ "subscription": "my-subscription" })
redis_list({ "subscription": "my-subscription" })

// Step 3: List all compute resources
aks_cluster_list({ "subscription": "my-subscription" })
functionapp_list({ "subscription": "my-subscription" })
appservice_list({ "subscription": "my-subscription" })

// Step 4: List all Key Vaults
keyvault_list({ "subscription": "my-subscription" })

// Step 5: Check quota usage
quota_list({
  "location": "eastus",
  "provider": "Microsoft.Compute"
})
```

## Cost Optimization Patterns

### Review Advisor Recommendations

```javascript
// Get cost optimization recommendations
advisor_recommendation_list({
  "category": "Cost"
})

// Get all recommendations for comprehensive review
advisor_recommendation_list({
  "subscription": "my-subscription"
})
```

### Common Cost Optimization Actions

| Finding | Action |
|---------|--------|
| Underutilized VMs | Right-size or deallocate |
| Idle databases | Scale down or pause |
| Unused storage | Delete or archive |
| Over-provisioned services | Reduce SKU tier |
| Missing reservations | Purchase reserved capacity |

## Troubleshooting Common Issues

| Issue | Diagnostic Steps |
|-------|------------------|
| Cannot list resources | Check subscription context with `az account show` |
| Access denied | Verify RBAC permissions with `role_assignment_list` |
| Resource not found | Confirm resource group and resource name |
| Quota exceeded | Check limits with `quota_list` |
| Slow operations | Review resource health and metrics |

## Related Steering Files

- **[security-guidelines.md](./security-guidelines.md)** - RBAC patterns, policy compliance, security best practices

## Resources

- [Azure Resource Naming Conventions](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
- [Azure Resource Organization](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources)
- [Azure Operational Excellence](https://docs.microsoft.com/azure/architecture/framework/devops/overview)
- [Azure Well-Architected Framework](https://docs.microsoft.com/azure/architecture/framework/)
- [Azure Advisor](https://docs.microsoft.com/azure/advisor/)

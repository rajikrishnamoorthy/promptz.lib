---
name: "azure-operations"
displayName: "Azure Operations"
description: "Manage Azure resources, enforce security policies, and control costs - storage, databases, containers, RBAC, and more"
keywords: ["azure", "storage", "cosmos", "keyvault", "sql", "aks", "redis", "rbac", "policy", "cost", "budget"]
author: "Volodymyr Marynychev"
---

# Azure Operations Power

## Quick Start

```bash
# 1. Verify prerequisites
node --version        # Need v20+
az --version          # Need Azure CLI installed

# 2. Authenticate with Azure
az login

# 3. Verify authentication
az account show       # Should display your subscription
```

After installing, try: *"List my storage accounts"* or *"Show Key Vault secrets in my subscription"*

## Overview

Manage and operate Azure resources with full read/write access. This power provides comprehensive resource management across storage, databases, containers, security, and governance.

**Key capabilities:**
- **Storage**: Manage storage accounts, blob containers, and data
- **Databases**: Work with Cosmos DB, SQL, PostgreSQL, MySQL, and Redis
- **Containers**: Manage AKS clusters, ACR registries, and container apps
- **Compute**: Manage Function Apps, App Services, and serverless workloads
- **Messaging**: Work with Event Hubs, Service Bus, Event Grid, and SignalR
- **Security**: Manage RBAC role assignments and Azure Policy compliance
- **Governance**: Monitor quotas and resource limits
- **Cost Optimization**: Get cost recommendations via Azure Advisor

**Authentication**: Requires `az login` with read/write permissions to your Azure subscription.

> **Note:** Cost management tools are available through Azure Advisor recommendations (`advisor_recommendation_list` with `category: "Cost"`). The Azure MCP provides cost optimization guidance via the advisor namespace rather than a separate costmanagement namespace.

> **Tip:** Use this power during the build and operate phases of your Azure projects. For design-time capabilities without authentication, use [azure-architect](../azure-architect/). For monitoring and troubleshooting, use [azure-monitoring](../azure-monitoring/).

## Available Steering Files

> **Note:** Steering files are accessed via the `readSteering` action in Kiro. Use `readSteering` with the power name and steering file name to retrieve detailed workflow guidance.

This power has the following steering files:
- **operations-best-practices.md** - Resource management patterns, naming conventions, operational excellence
- **security-guidelines.md** - RBAC patterns, policy compliance, security best practices, least privilege access

## Available MCP Servers

### azure-mcp
**Package:** `@azure/mcp` | **Connection:** npx stdio

#### Configuration

```json
{
  "mcpServers": {
    "azure-mcp": {
      "command": "npx",
      "args": [
        "-y", "@azure/mcp@latest", "server", "start",
        "--namespace", "storage",
        "--namespace", "keyvault",
        "--namespace", "cosmos",
        "--namespace", "sql",
        "--namespace", "postgres",
        "--namespace", "mysql",
        "--namespace", "redis",
        "--namespace", "aks",
        "--namespace", "acr",
        "--namespace", "functionapp",
        "--namespace", "appservice",
        "--namespace", "eventhubs",
        "--namespace", "servicebus",
        "--namespace", "eventgrid",
        "--namespace", "signalr",
        "--namespace", "role",
        "--namespace", "policy",
        "--namespace", "advisor",
        "--namespace", "quota"
      ],
      "env": {
        "AZURE_MCP_COLLECT_TELEMETRY": "false"
      }
    }
  }
}
```

---

## Storage Tools

### azmcp_storage_account_list
Lists all storage accounts in a subscription or resource group.
- Optional: `subscription` (string) - Azure subscription ID or name
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of storage accounts with names, locations, SKUs, and access tiers

**Example:**
```javascript
azmcp_storage_account_list({
  "subscription": "my-subscription",
  "resource-group": "my-rg"
})
```

### azmcp_storage_container_list
Lists blob containers in a storage account.
- Required: `account-name` (string) - Storage account name
- Optional: `subscription` (string) - Azure subscription ID
- Returns: List of containers with names, public access levels, and metadata

**Example:**
```javascript
azmcp_storage_container_list({
  "account-name": "mystorageaccount"
})
```

### azmcp_storage_blob_list
Lists blobs in a storage container.
- Required: `account-name` (string) - Storage account name
- Required: `container-name` (string) - Container name
- Optional: `prefix` (string) - Filter blobs by prefix
- Returns: List of blobs with names, sizes, content types, and last modified dates

**Example:**
```javascript
azmcp_storage_blob_list({
  "account-name": "mystorageaccount",
  "container-name": "documents",
  "prefix": "reports/"
})
```

---

## Key Vault Tools

### azmcp_keyvault_list
Lists all Key Vaults in a subscription or resource group.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Key Vaults with names, locations, and SKUs

**Example:**
```javascript
azmcp_keyvault_list({
  "resource-group": "security-rg"
})
```

### azmcp_keyvault_secret_list
Lists secrets in a Key Vault (names only, not values).
- Required: `vault-name` (string) - Key Vault name
- Returns: List of secret names with enabled status and expiration dates

**Example:**
```javascript
azmcp_keyvault_secret_list({
  "vault-name": "my-keyvault"
})
```

### azmcp_keyvault_key_list
Lists cryptographic keys in a Key Vault.
- Required: `vault-name` (string) - Key Vault name
- Returns: List of keys with names, types, and enabled status

**Example:**
```javascript
azmcp_keyvault_key_list({
  "vault-name": "my-keyvault"
})
```

---

## Cosmos DB Tools

### azmcp_cosmos_account_list
Lists Cosmos DB accounts in a subscription or resource group.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Cosmos DB accounts with names, locations, and API types

**Example:**
```javascript
azmcp_cosmos_account_list({
  "resource-group": "data-rg"
})
```

### azmcp_cosmos_database_list
Lists databases in a Cosmos DB account.
- Required: `account-name` (string) - Cosmos DB account name
- Required: `resource-group` (string) - Resource group name
- Returns: List of databases with names and throughput settings

**Example:**
```javascript
azmcp_cosmos_database_list({
  "account-name": "my-cosmos",
  "resource-group": "data-rg"
})
```

### azmcp_cosmos_query
Executes a SQL query against a Cosmos DB container.
- Required: `account-name` (string) - Cosmos DB account name
- Required: `database-name` (string) - Database name
- Required: `container-name` (string) - Container name
- Required: `query` (string) - SQL query to execute
- Optional: `resource-group` (string) - Resource group name
- Returns: Query results as JSON documents

**Example:**
```javascript
azmcp_cosmos_query({
  "account-name": "my-cosmos",
  "database-name": "mydb",
  "container-name": "items",
  "query": "SELECT * FROM c WHERE c.status = 'active' LIMIT 10"
})
```

---

## SQL Database Tools

### azmcp_sql_server_list
Lists Azure SQL servers in a subscription or resource group.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of SQL servers with names, locations, and admin logins

**Example:**
```javascript
azmcp_sql_server_list({
  "resource-group": "data-rg"
})
```

### azmcp_sql_database_list
Lists databases on an Azure SQL server.
- Required: `server-name` (string) - SQL server name
- Required: `resource-group` (string) - Resource group name
- Returns: List of databases with names, SKUs, and sizes

**Example:**
```javascript
azmcp_sql_database_list({
  "server-name": "my-sql-server",
  "resource-group": "data-rg"
})
```

---

## PostgreSQL Tools

### azmcp_postgres_server_list
Lists Azure Database for PostgreSQL Flexible Servers.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of PostgreSQL servers with names, versions, and SKUs

**Example:**
```javascript
azmcp_postgres_server_list({
  "resource-group": "data-rg"
})
```

### azmcp_postgres_database_list
Lists databases on a PostgreSQL Flexible Server.
- Required: `server-name` (string) - PostgreSQL server name
- Required: `resource-group` (string) - Resource group name
- Returns: List of databases with names and character sets

**Example:**
```javascript
azmcp_postgres_database_list({
  "server-name": "my-postgres",
  "resource-group": "data-rg"
})
```

---

## MySQL Tools

### azmcp_mysql_server_list
Lists Azure Database for MySQL Flexible Servers.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of MySQL servers with names, versions, and SKUs

**Example:**
```javascript
azmcp_mysql_server_list({
  "resource-group": "data-rg"
})
```

### azmcp_mysql_database_list
Lists databases on a MySQL Flexible Server.
- Required: `server-name` (string) - MySQL server name
- Required: `resource-group` (string) - Resource group name
- Returns: List of databases with names and character sets

**Example:**
```javascript
azmcp_mysql_database_list({
  "server-name": "my-mysql",
  "resource-group": "data-rg"
})
```

---

## Redis Tools

### azmcp_redis_list
Lists Azure Managed Redis and Azure Cache for Redis instances.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Redis instances with names, SKUs, and connection details

**Example:**
```javascript
azmcp_redis_list({
  "resource-group": "cache-rg"
})
```

---

## AKS Tools

### azmcp_aks_cluster_list
Lists Azure Kubernetes Service clusters.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of AKS clusters with names, versions, and node counts

**Example:**
```javascript
azmcp_aks_cluster_list({
  "resource-group": "k8s-rg"
})
```

### azmcp_aks_nodepool_list
Lists node pools in an AKS cluster.
- Required: `cluster-name` (string) - AKS cluster name
- Required: `resource-group` (string) - Resource group name
- Returns: List of node pools with names, VM sizes, and node counts

**Example:**
```javascript
azmcp_aks_nodepool_list({
  "cluster-name": "my-aks",
  "resource-group": "k8s-rg"
})
```

---

## ACR Tools

### azmcp_acr_list
Lists Azure Container Registries.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of container registries with names, SKUs, and login servers

**Example:**
```javascript
azmcp_acr_list({
  "resource-group": "containers-rg"
})
```

---

## Function App Tools

### azmcp_functionapp_list
Lists Azure Function Apps.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Function Apps with names, runtimes, and states

**Example:**
```javascript
azmcp_functionapp_list({
  "resource-group": "serverless-rg"
})
```

---

## App Service Tools

### azmcp_appservice_list
Lists Azure App Services (Web Apps).
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of App Services with names, plans, and states

**Example:**
```javascript
azmcp_appservice_list({
  "resource-group": "web-rg"
})
```

---

## Event Hubs Tools

### azmcp_eventhubs_namespace_list
Lists Event Hubs namespaces.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Event Hubs namespaces with names, SKUs, and throughput units

**Example:**
```javascript
azmcp_eventhubs_namespace_list({
  "resource-group": "messaging-rg"
})
```

---

## Service Bus Tools

### azmcp_servicebus_namespace_list
Lists Service Bus namespaces.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Service Bus namespaces with names, SKUs, and messaging units

**Example:**
```javascript
azmcp_servicebus_namespace_list({
  "resource-group": "messaging-rg"
})
```

---

## Event Grid Tools

### azmcp_eventgrid_topic_list
Lists Event Grid topics.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of Event Grid topics with names and endpoints

**Example:**
```javascript
azmcp_eventgrid_topic_list({
  "resource-group": "events-rg"
})
```

---

## SignalR Tools

### azmcp_signalr_list
Lists Azure SignalR Service instances.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `resource-group` (string) - Filter by resource group
- Returns: List of SignalR services with names, SKUs, and connection strings

**Example:**
```javascript
azmcp_signalr_list({
  "resource-group": "realtime-rg"
})
```

---

## RBAC Tools

### azmcp_role_assignment_list
Lists role assignments at a specified scope.
- Optional: `scope` (string) - Scope for role assignments (subscription, resource group, or resource)
- Optional: `assignee` (string) - Filter by assignee (user, group, or service principal)
- Returns: List of role assignments with principals, roles, and scopes

**Example:**
```javascript
azmcp_role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-rg"
})
```

---

## Policy Tools

### azmcp_policy_assignment_list
Lists Azure Policy assignments.
- Optional: `scope` (string) - Scope for policy assignments
- Optional: `subscription` (string) - Azure subscription ID
- Returns: List of policy assignments with names, definitions, and enforcement modes

**Example:**
```javascript
azmcp_policy_assignment_list({
  "subscription": "my-subscription"
})
```

---

## Advisor Tools

### azmcp_advisor_recommendation_list
Lists Azure Advisor recommendations.
- Optional: `subscription` (string) - Azure subscription ID
- Optional: `category` (string) - Filter by category (Cost, Security, Reliability, Performance, OperationalExcellence)
- Returns: List of recommendations with impact, category, and remediation steps

**Example:**
```javascript
azmcp_advisor_recommendation_list({
  "category": "Cost"
})
```

---

## Quota Tools

### azmcp_quota_list
Lists quota usage for Azure resources.
- Required: `location` (string) - Azure region
- Optional: `provider` (string) - Resource provider (e.g., "Microsoft.Compute")
- Returns: Current usage and limits for resources in the specified region

**Example:**
```javascript
azmcp_quota_list({
  "location": "eastus",
  "provider": "Microsoft.Compute"
})
```

### azmcp_quota_regions_list
Lists available Azure regions for a resource type.
- Required: `resource-type` (string) - Azure resource type
- Returns: List of regions where the resource type is available

**Example:**
```javascript
azmcp_quota_regions_list({
  "resource-type": "Microsoft.Compute/virtualMachines"
})
```

---

## Workflow Examples

### Managing Storage Resources

```javascript
// Step 1: List storage accounts
azmcp_storage_account_list({
  "resource-group": "data-rg"
})

// Step 2: List containers in a storage account
azmcp_storage_container_list({
  "account-name": "mystorageaccount"
})

// Step 3: List blobs in a container
azmcp_storage_blob_list({
  "account-name": "mystorageaccount",
  "container-name": "documents"
})
```

### Database Operations

```javascript
// Step 1: List Cosmos DB accounts
azmcp_cosmos_account_list({
  "resource-group": "data-rg"
})

// Step 2: List databases
azmcp_cosmos_database_list({
  "account-name": "my-cosmos",
  "resource-group": "data-rg"
})

// Step 3: Query data
azmcp_cosmos_query({
  "account-name": "my-cosmos",
  "database-name": "mydb",
  "container-name": "orders",
  "query": "SELECT * FROM c WHERE c.status = 'pending'"
})
```

### Security and RBAC Audit

```javascript
// Step 1: List role assignments for a resource group
azmcp_role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/production-rg"
})

// Step 2: Check policy compliance
azmcp_policy_assignment_list({
  "subscription": "my-subscription"
})

// Step 3: Review security recommendations
azmcp_advisor_recommendation_list({
  "category": "Security"
})
```

### Container Infrastructure Review

```javascript
// Step 1: List AKS clusters
azmcp_aks_cluster_list({
  "resource-group": "k8s-rg"
})

// Step 2: Review node pools
azmcp_aks_nodepool_list({
  "cluster-name": "production-aks",
  "resource-group": "k8s-rg"
})

// Step 3: List container registries
azmcp_acr_list({
  "resource-group": "containers-rg"
})
```

### Cost Optimization

```javascript
// Step 1: Get cost recommendations
azmcp_advisor_recommendation_list({
  "category": "Cost"
})

// Step 2: Check quota usage
azmcp_quota_list({
  "location": "eastus",
  "provider": "Microsoft.Compute"
})

// Step 3: Review available regions for optimization
azmcp_quota_regions_list({
  "resource-type": "Microsoft.Compute/virtualMachines"
})
```

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| `Authentication required` | Run `az login` and ensure you have appropriate permissions |
| `Subscription not found` | Verify subscription ID with `az account list` |
| `Resource group not found` | Check resource group name and subscription context |
| `Access denied` | Verify RBAC permissions for the resource |
| `Resource not found` | Confirm resource exists in the specified location |

## Related Powers

| Power | Use Case |
|-------|----------|
| [azure-architect](../azure-architect/) | Design infrastructure before deploying with this power |
| [azure-monitoring](../azure-monitoring/) | Monitor and troubleshoot resources managed by this power |

## Best Practices

- **Use resource groups** - Always scope operations to specific resource groups when possible
- **Follow least privilege** - Use the minimum permissions needed for operations
- **Check Advisor regularly** - Review recommendations for cost, security, and performance
- **Audit RBAC assignments** - Regularly review role assignments for compliance
- **Monitor quotas** - Check quota usage before scaling resources
- **Use naming conventions** - Follow consistent naming for easier resource management

## Resources

- [Azure MCP Server Documentation](https://learn.microsoft.com/azure/developer/azure-mcp-server/)
- [Microsoft MCP Repository](https://github.com/microsoft/mcp) - Source code
- [Azure Resource Management](https://docs.microsoft.com/azure/azure-resource-manager/)
- [Azure RBAC Documentation](https://docs.microsoft.com/azure/role-based-access-control/)
- [Azure Policy Documentation](https://docs.microsoft.com/azure/governance/policy/)
- [Azure Advisor](https://docs.microsoft.com/azure/advisor/)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)

---

**Package:** `@azure/mcp` | **License:** MIT

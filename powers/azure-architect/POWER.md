---
name: "azure-architect"
displayName: "Azure Architect"
description: "Design Azure infrastructure with official documentation, best practices, and architecture guidance - no authentication required"
keywords: ["bicep", "arm", "iac", "infrastructure", "architecture", "vnet", "network-design", "deployment", "azd", "azure-developer-cli", "azure", "cloud", "design"]
author: "Volodymyr Marynychev"
---

# Azure Architect Power

## Quick Start

No authentication or setup required. All tools work immediately without Azure login.

```bash
# Optional: Verify Node.js for Azure MCP server
node --version  # Need v20+
```

After installing, try:
- *"Design a microservices architecture for e-commerce"*
- *"Get Bicep schema for storage account"*
- *"Search Azure docs for App Service best practices"*
- *"What are Azure best practices for Functions?"*

## Overview

Design and plan Azure infrastructure using official Microsoft documentation, best practices, and architecture guidance. This power focuses on **design-time** capabilities that require **no Azure authentication**.

**Key capabilities:**
- **Documentation**: Search and fetch official Azure documentation from Microsoft Learn
- **Bicep Schemas**: Get resource schemas for Infrastructure as Code
- **Architecture Design**: Interactive cloud architecture recommendations
- **Best Practices**: Azure and Terraform best practices for code generation/deployment
- **IaC Guidance**: Deployment rules and CI/CD pipeline guidance

**Authentication**: None required. All tools work without any Azure login.

> **Tip:** Use this power during the design phase. For deploying resources, use [azure-operations](../azure-operations/). For monitoring, use [azure-monitoring](../azure-monitoring/).

## Available Steering Files

- **iac-workflows.md** - Bicep/ARM patterns, deployment workflows, CI/CD guidance
- **networking-patterns.md** - Network design patterns, NSG rules, private endpoints, hub-spoke topologies

## MCP Server Configuration

```json
{
  "mcpServers": {
    "microsoft-docs": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    },
    "azure-mcp": {
      "command": "npx",
      "args": [
        "-y", "@azure/mcp@latest", "server", "start",
        "--namespace", "documentation",
        "--namespace", "bicepschema",
        "--namespace", "cloudarchitect",
        "--namespace", "bestpractices"
      ],
      "env": {
        "AZURE_MCP_COLLECT_TELEMETRY": "false"
      }
    }
  }
}
```

## Available MCP Tools

### microsoft-docs Server (Remote HTTP)

| Tool | Description |
|------|-------------|
| `microsoft_docs_search` | Search official Azure documentation |
| `microsoft_docs_fetch` | Fetch full content of documentation articles |
| `microsoft_code_sample_search` | Find Azure code samples |

### azure-mcp Server (Local)

| Tool | Description |
|------|-------------|
| `documentation` | Search/fetch Microsoft docs (alternative to microsoft-docs) |
| `bicepschema` | Get Bicep schemas for Azure resource types |
| `cloudarchitect` | Interactive architecture design recommendations |
| `get_azure_bestpractices` | Azure best practices for code generation/deployment |
| `azureterraformbestpractices` | Terraform best practices for Azure |

> **Tip:** Use `learn=true` parameter on hierarchical tools to discover available subcommands.

## Workflow Examples

### Design Cloud Architecture (Interactive)

```javascript
// Start architecture design session
cloudarchitect({
  intent: "Design e-commerce architecture",
  command: "cloudarchitect_design",
  parameters: {}
})
// Returns: Guided questions to gather requirements
// Continue the conversation to refine the architecture
```

### Get Bicep Schema for Resource

```javascript
// Get schema for storage account
bicepschema({
  intent: "Get storage account schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Storage/storageAccounts" }
})

// Get schema for App Service
bicepschema({
  intent: "Get App Service schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Web/sites" }
})
```

### Get Best Practices

```javascript
// Azure general best practices
get_azure_bestpractices({
  intent: "Get Azure best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "all" }
})

// Azure Functions best practices
get_azure_bestpractices({
  intent: "Get Functions best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "azurefunctions", action: "code-generation" }
})

// Terraform best practices for Azure
azureterraformbestpractices({
  intent: "Get Terraform best practices",
  command: "azureterraformbestpractices_get",
  parameters: {}
})
```

### Search → Fetch Documentation Pattern

For complete documentation, use the search → fetch pattern:

```javascript
// Step 1: Search to find relevant articles
microsoft_docs_search({ query: "Azure Monitor metric alerts" })

// Step 2: Fetch full content for articles you need
microsoft_docs_fetch({
  url: "https://learn.microsoft.com/azure/azure-monitor/alerts/action-groups"
})
```

### Designing a New Azure Resource

```javascript
// Step 1: Get Bicep schema
bicepschema({
  intent: "Get App Service schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Web/sites" }
})

// Step 2: Get best practices
get_azure_bestpractices({
  intent: "Get best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "code-generation" }
})

// Step 3: Search for documentation
microsoft_docs_search({ query: "Azure App Service configuration best practices" })

// Step 4: Find code samples
microsoft_code_sample_search({ query: "App Service Bicep template" })
```

### Planning Infrastructure Architecture

```javascript
// Use interactive architecture design
cloudarchitect({
  intent: "Design microservices architecture",
  command: "cloudarchitect_design",
  parameters: {
    question: "What type of application are you building?",
    "question-number": 1,
    "total-questions": 5,
    "next-question-needed": true
  }
})

// Or search for reference architectures
microsoft_docs_search({ query: "Azure reference architecture web application" })
```

### Writing Bicep/Terraform Code

```javascript
// Get Bicep schema for the resource
bicepschema({
  intent: "Get Key Vault schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.KeyVault/vaults" }
})

// Get best practices
get_azure_bestpractices({
  intent: "Get code generation best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "code-generation" }
})

// For Terraform, get best practices first
azureterraformbestpractices({
  intent: "Get Terraform best practices",
  command: "azureterraformbestpractices_get",
  parameters: {}
})
```

### Network Architecture Design

```javascript
// Search for network design patterns
microsoft_docs_search({ query: "hub-spoke network topology Azure" })

// Get VNet schema
bicepschema({
  intent: "Get VNet schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Network/virtualNetworks" }
})
```

## Common Bicep Resource Types

| Resource | Type String |
|----------|-------------|
| Storage Account | `Microsoft.Storage/storageAccounts` |
| App Service | `Microsoft.Web/sites` |
| Function App | `Microsoft.Web/sites` (kind: functionapp) |
| Key Vault | `Microsoft.KeyVault/vaults` |
| Virtual Network | `Microsoft.Network/virtualNetworks` |
| SQL Database | `Microsoft.Sql/servers/databases` |
| Cosmos DB | `Microsoft.DocumentDB/databaseAccounts` |
| Container App | `Microsoft.App/containerApps` |
| AKS Cluster | `Microsoft.ContainerService/managedClusters` |

## Effective Search Tips

### Be Specific
Instead of: `"Azure storage"`
Use: `"Azure blob storage lifecycle management policies"`

### Use Service Names
Instead of: `"database"`
Use: `"Azure SQL Database security best practices"`

## Common Search Queries

| Topic | Recommended Search Query |
|-------|-------------------------|
| **Compute** | `"Azure App Service deployment slots best practices"` |
| **Containers** | `"AKS cluster networking CNI vs kubenet"` |
| **Storage** | `"Azure Storage redundancy options comparison"` |
| **Databases** | `"Azure SQL Database high availability configuration"` |
| **Networking** | `"Azure virtual network subnet planning"` |
| **Security** | `"Azure Key Vault access policies vs RBAC"` |
| **Identity** | `"Azure AD managed identity authentication"` |
| **Monitoring** | `"Azure Monitor alerts best practices"` |
| **IaC** | `"Bicep vs ARM templates comparison"` |
| **DevOps** | `"Azure DevOps pipeline YAML syntax"` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `No results found` | Try broader search terms or check spelling |
| `Article not found` | Verify the URL is correct and the article exists |
| `Azure MCP not loading` | Ensure Node.js v20+ is installed |
| `Bicep schema not found` | Check resource type format: `Microsoft.Provider/resourceType` |

## Related Powers

| Power | Use Case |
|-------|----------|
| [azure-operations](../azure-operations/) | Deploy and manage the resources you've designed |
| [azure-monitoring](../azure-monitoring/) | Monitor and troubleshoot deployed infrastructure |

## Best Practices

- **Start with architecture design** - Use `cloudarchitect` for guided architecture recommendations
- **Get schemas before coding** - Use `bicepschema` to understand resource properties
- **Check best practices first** - Use `get_azure_bestpractices` before generating code
- **Verify with official docs** - Use `microsoft_docs_fetch` for complete documentation
- **Use steering files** - Read `iac-workflows.md` and `networking-patterns.md` for detailed patterns

## Resources

- [Microsoft Learn - Azure Documentation](https://learn.microsoft.com/azure/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Azure Well-Architected Framework](https://docs.microsoft.com/azure/architecture/framework/)
- [Bicep Documentation](https://docs.microsoft.com/azure/azure-resource-manager/bicep/)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates)

---

**MCP Servers:** Microsoft Learn, Azure MCP | **Package:** `@azure/mcp` | **License:** MIT

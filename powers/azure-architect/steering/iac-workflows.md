# IaC Workflows Guide

This steering file provides guidance for Infrastructure as Code (IaC) tool selection, Bicep/ARM patterns, and deployment workflows using the azure-architect power.

## Using Azure MCP Tools for IaC

The azure-architect power includes Azure MCP tools that work without authentication:

### Get Bicep Schemas

Use `bicepschema` to get the exact schema for any Azure resource:

```javascript
// Get storage account schema
bicepschema({
  intent: "Get storage account schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Storage/storageAccounts" }
})

// Get App Service schema
bicepschema({
  intent: "Get App Service schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Web/sites" }
})

// Get Key Vault schema
bicepschema({
  intent: "Get Key Vault schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.KeyVault/vaults" }
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

// Code generation best practices
get_azure_bestpractices({
  intent: "Get code generation best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "code-generation" }
})

// Deployment best practices
get_azure_bestpractices({
  intent: "Get deployment best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "deployment" }
})

// Terraform best practices for Azure
azureterraformbestpractices({
  intent: "Get Terraform best practices",
  command: "azureterraformbestpractices_get",
  parameters: {}
})
```

## Handling Truncated Documentation

Search results from `microsoft_docs_search` return excerpts that may be truncated. Follow this pattern for complete information:

```javascript
// Step 1: Search to find relevant articles
microsoft_docs_search({ query: "Bicep storage account properties" })

// Step 2: If excerpt shows "..." or seems incomplete, fetch full article
microsoft_docs_fetch({
  url: "https://learn.microsoft.com/azure/azure-resource-manager/bicep/..."
})
```

## Tool Selection Decision Tree

Use this decision tree to choose the right IaC tool for your project:

```
┌─────────────────────────────────────────────────────────────────┐
│                    IaC Tool Selection                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │ Azure-only infrastructure?    │
              └───────────────────────────────┘
                     │              │
                    Yes             No
                     │              │
                     ▼              ▼
         ┌─────────────────┐  ┌─────────────────┐
         │ Team experience │  │ Use Terraform   │
         │ with ARM/Bicep? │  │ (multi-cloud)   │
         └─────────────────┘  └─────────────────┘
              │        │
             Yes       No
              │        │
              ▼        ▼
    ┌──────────────┐  ┌──────────────────────┐
    │ Use Bicep    │  │ Existing Terraform   │
    │ (native)     │  │ expertise?           │
    └──────────────┘  └──────────────────────┘
                            │        │
                           Yes       No
                            │        │
                            ▼        ▼
                  ┌──────────────┐  ┌──────────────┐
                  │ Use Terraform│  │ Use Bicep    │
                  │              │  │ (easier)     │
                  └──────────────┘  └──────────────┘
```

### Quick Comparison

| Factor | Bicep | Terraform |
|--------|-------|-----------|
| **Learning curve** | Lower (Azure-native) | Higher (HCL syntax) |
| **Multi-cloud** | No | Yes |
| **State management** | Azure handles it | You manage state |
| **Module ecosystem** | Azure Verified Modules | Terraform Registry |
| **IDE support** | VS Code extension | Multiple IDEs |
| **Preview features** | Day-0 support | Depends on provider |

## Bicep Patterns

### Getting Started with Bicep

```javascript
// Step 1: Get best practices first
get_azure_bestpractices({
  intent: "Get Azure best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "code-generation" }
})

// Step 2: Get schema for your resource
bicepschema({
  intent: "Get storage account schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Storage/storageAccounts" }
})

// Step 3: Search for additional documentation
microsoft_docs_search({ query: "Bicep file structure best practices Azure" })
```

### Bicep Module Structure

Organize your Bicep code using this recommended structure:

```
infra/
├── main.bicep              # Entry point, orchestrates modules
├── main.bicepparam         # Parameter file for deployment
├── modules/
│   ├── networking/
│   │   ├── vnet.bicep
│   │   └── nsg.bicep
│   ├── compute/
│   │   ├── appservice.bicep
│   │   └── functions.bicep
│   └── data/
│       ├── storage.bicep
│       └── cosmos.bicep
└── environments/
    ├── dev.bicepparam
    ├── staging.bicepparam
    └── prod.bicepparam
```

### Bicep Best Practices

1. **Use parameter files** - Separate configuration from code
2. **Create reusable modules** - One resource type per module
3. **Use Azure Verified Modules** - Leverage community-tested modules
4. **Enable linting** - Use `bicep build` with linter rules
5. **Use `@description` decorators** - Document parameters and outputs

### Common Bicep Patterns

#### Resource Naming Convention
```bicep
// Use a naming module for consistency
param environment string
param location string = resourceGroup().location
param workloadName string

var resourceToken = toLower(uniqueString(subscription().id, resourceGroup().id))
var abbrs = loadJsonContent('abbreviations.json')

// Example: st-myapp-dev-abc123
var storageAccountName = '${abbrs.storage}${workloadName}${environment}${resourceToken}'
```

#### Conditional Deployment
```bicep
param deployRedis bool = false

resource redis 'Microsoft.Cache/redis@2023-08-01' = if (deployRedis) {
  name: redisName
  location: location
  properties: {
    sku: {
      name: 'Basic'
      family: 'C'
      capacity: 0
    }
  }
}
```

#### Loops for Multiple Resources
```bicep
param storageAccounts array = [
  { name: 'logs', sku: 'Standard_LRS' }
  { name: 'data', sku: 'Standard_GRS' }
]

resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = [for account in storageAccounts: {
  name: '${account.name}${resourceToken}'
  location: location
  sku: { name: account.sku }
  kind: 'StorageV2'
}]
```

### Finding Bicep Resource Schemas

Use the `bicepschema` tool to get exact schemas:

```javascript
// Get schema for any Azure resource type
bicepschema({
  intent: "Get resource schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Storage/storageAccounts" }
})
```

Common resource types:
- `Microsoft.Storage/storageAccounts`
- `Microsoft.Web/sites`
- `Microsoft.KeyVault/vaults`
- `Microsoft.Network/virtualNetworks`
- `Microsoft.Sql/servers/databases`
- `Microsoft.DocumentDB/databaseAccounts`
- `Microsoft.App/containerApps`
- `Microsoft.ContainerService/managedClusters`

## Terraform Patterns

### Getting Started with Terraform

```javascript
// Step 1: Get Terraform best practices for Azure
azureterraformbestpractices({
  intent: "Get Terraform best practices",
  command: "azureterraformbestpractices_get",
  parameters: {}
})

// Step 2: Search for additional documentation
microsoft_docs_search({ query: "Terraform Azure provider getting started" })
```

### Terraform Module Structure

```
terraform/
├── main.tf                 # Root module, calls child modules
├── variables.tf            # Input variables
├── outputs.tf              # Output values
├── providers.tf            # Provider configuration
├── versions.tf             # Version constraints
├── terraform.tfvars        # Variable values (gitignored)
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   └── data/
└── environments/
    ├── dev/
    │   └── terraform.tfvars
    ├── staging/
    └── prod/
```

### Terraform State Management

**Recommended: Azure Storage Backend**

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

**State Management Best Practices:**
1. **Use remote state** - Never store state locally in production
2. **Enable state locking** - Azure Storage provides this automatically
3. **Separate state per environment** - Use different state files for dev/staging/prod
4. **Enable versioning** - Turn on blob versioning for state recovery
5. **Restrict access** - Use RBAC to limit who can modify state

### Common Terraform Patterns

#### Resource Naming with Random Suffix
```hcl
resource "random_string" "suffix" {
  length  = 6
  special = false
  upper   = false
}

locals {
  resource_suffix = random_string.suffix.result
  storage_name    = "st${var.workload}${var.environment}${local.resource_suffix}"
}
```

#### Conditional Resources with count
```hcl
variable "deploy_redis" {
  type    = bool
  default = false
}

resource "azurerm_redis_cache" "main" {
  count               = var.deploy_redis ? 1 : 0
  name                = "redis-${var.workload}-${var.environment}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  capacity            = 0
  family              = "C"
  sku_name            = "Basic"
}
```

#### Dynamic Blocks for Complex Configuration
```hcl
resource "azurerm_network_security_group" "main" {
  name                = "nsg-${var.workload}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  dynamic "security_rule" {
    for_each = var.nsg_rules
    content {
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = security_rule.value.direction
      access                     = security_rule.value.access
      protocol                   = security_rule.value.protocol
      source_port_range          = security_rule.value.source_port_range
      destination_port_range     = security_rule.value.destination_port_range
      source_address_prefix      = security_rule.value.source_address_prefix
      destination_address_prefix = security_rule.value.destination_address_prefix
    }
  }
}
```

## Deployment Workflows

### Planning a Deployment

```javascript
// Step 1: Search for architecture patterns
microsoft_docs_search({
  "query": "Azure architecture web application database caching"
})

// Step 2: Get deployment best practices
microsoft_docs_search({
  "query": "Azure deployment best practices CI/CD"
})

// Step 3: Find pipeline guidance
microsoft_docs_search({
  "query": "GitHub Actions Azure deployment workflow"
})
```

### CI/CD Pipeline Patterns

#### GitHub Actions Workflow Structure

```yaml
name: Deploy Infrastructure

on:
  push:
    branches: [main]
    paths: ['infra/**']
  pull_request:
    branches: [main]
    paths: ['infra/**']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Bicep Build (Validate)
        run: az bicep build --file infra/main.bicep

  plan:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - name: What-If Deployment
        run: |
          az deployment sub what-if \
            --location eastus \
            --template-file infra/main.bicep \
            --parameters infra/environments/${{ github.event.inputs.environment }}.bicepparam

  deploy:
    needs: plan
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - name: Deploy Infrastructure
        run: |
          az deployment sub create \
            --location eastus \
            --template-file infra/main.bicep \
            --parameters infra/environments/prod.bicepparam
```

### Environment Promotion Strategy

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│     Dev     │ ──▶ │   Staging   │ ──▶ │ Production  │
│             │     │             │     │             │
│ Auto-deploy │     │ Auto-deploy │     │ Manual gate │
│ on PR merge │     │ after dev   │     │ + approval  │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Best Practices:**
1. **Use identical IaC** - Same templates across all environments
2. **Parameterize differences** - Environment-specific values in parameter files
3. **Implement gates** - Require approvals for production
4. **Run what-if/plan** - Always preview changes before applying
5. **Use deployment slots** - For zero-downtime application deployments

## Common Workflows

### New Project Setup

1. **Choose your IaC tool** using the decision tree above
2. **Search for best practices**:
   ```javascript
   // For Bicep
   microsoft_docs_search({ "query": "Bicep project structure best practices" })
   
   // For Terraform
   microsoft_docs_search({ "query": "Terraform Azure module structure" })
   ```
3. **Find architecture patterns**:
   ```javascript
   microsoft_docs_search({
     "query": "Azure reference architecture your-workload-type"
   })
   ```
4. **Get code samples** for each resource type you need
5. **Search for deployment patterns**:
   ```javascript
   microsoft_docs_search({ "query": "Azure deployment CI/CD best practices" })
   ```
6. **Set up CI/CD** using pipeline guidance

### Adding a New Resource

1. **Search for resource documentation**:
   ```javascript
   microsoft_docs_search({ "query": "Azure resource-type configuration best practices" })
   ```
2. **Find code samples**:
   ```javascript
   microsoft_code_sample_search({ "query": "Bicep resource-type example" })
   ```
3. **Get ARM template reference** (includes all properties):
   ```javascript
   microsoft_docs_search({ "query": "resource-type ARM template reference" })
   ```
4. **Add to your IaC** following module patterns above
5. **Test with what-if/plan** before deploying

### Migrating from ARM to Bicep

1. **Decompile existing ARM templates**:
   ```bash
   az bicep decompile --file template.json
   ```
2. **Search for migration guidance**:
   ```javascript
   microsoft_docs_search({ "query": "ARM to Bicep migration guide" })
   ```
3. **Get Bicep best practices**:
   ```javascript
   microsoft_docs_search({ "query": "Bicep best practices modules" })
   ```
4. **Refactor into modules** following the structure above
5. **Validate with bicep build**
6. **Test deployment** in a non-production environment

## Related Steering Files

- **[networking-patterns.md](./networking-patterns.md)** - Network design patterns, NSG rules, private endpoints

## Resources

- [Bicep Documentation](https://docs.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure Verified Modules](https://azure.github.io/Azure-Verified-Modules/)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure Deployment Environments](https://docs.microsoft.com/azure/deployment-environments/)
- [ARM Template Reference](https://docs.microsoft.com/azure/templates/)

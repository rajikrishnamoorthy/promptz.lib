# Networking Patterns Guide

This steering file provides guidance for Azure network design patterns, NSG rules, private endpoints, and hub-spoke topologies using the azure-architect power.

## Using Azure MCP Tools for Networking

### Get Network Resource Schemas

Use `bicepschema` to get exact schemas for network resources:

```javascript
// Get VNet schema
bicepschema({
  intent: "Get VNet schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Network/virtualNetworks" }
})

// Get NSG schema
bicepschema({
  intent: "Get NSG schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Network/networkSecurityGroups" }
})

// Get Private Endpoint schema
bicepschema({
  intent: "Get Private Endpoint schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Network/privateEndpoints" }
})

// Get Application Gateway schema
bicepschema({
  intent: "Get App Gateway schema",
  command: "bicepschema_get",
  parameters: { "resource-type": "Microsoft.Network/applicationGateways" }
})
```

### Design Network Architecture

Use `cloudarchitect` for interactive architecture design:

```javascript
cloudarchitect({
  intent: "Design hub-spoke network architecture",
  command: "cloudarchitect_design",
  parameters: {}
})
```

### Get Best Practices

```javascript
get_azure_bestpractices({
  intent: "Get Azure networking best practices",
  command: "get_azure_bestpractices_get",
  parameters: { resource: "general", action: "all" }
})
```

## Handling Truncated Documentation

Network documentation often contains detailed tables, IP ranges, and configuration specifics that get truncated in search excerpts. Always fetch full articles for:

- **NSG rule tables** - Complete priority lists and port ranges
- **Private DNS zone names** - Full list of zones per service
- **Subnet sizing** - Exact CIDR requirements and reserved IPs
- **Service tag lists** - Complete tag names and descriptions

```javascript
// Example: Get complete private DNS zone list
microsoft_docs_search({
  "query": "private endpoint DNS zone names Azure"
})
// Excerpt may show only a few zones...

// Fetch full article for complete list
microsoft_docs_fetch({
  "url": "https://learn.microsoft.com/azure/private-link/private-endpoint-dns"
})
```

## Network Architecture Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                 Network Architecture Selection                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │ Single workload or multiple?  │
              └───────────────────────────────┘
                     │              │
                  Single        Multiple
                     │              │
                     ▼              ▼
         ┌─────────────────┐  ┌─────────────────────┐
         │ Simple VNet     │  │ Need centralized    │
         │ with subnets    │  │ connectivity?       │
         └─────────────────┘  └─────────────────────┘
                                   │        │
                                  Yes       No
                                   │        │
                                   ▼        ▼
                        ┌──────────────┐  ┌──────────────────┐
                        │ Hub-Spoke    │  │ VNet Peering     │
                        │ topology     │  │ (mesh)           │
                        └──────────────┘  └──────────────────┘
```

## Getting Network Documentation

```javascript
// Search for VNet documentation
microsoft_docs_search({ query: "Azure virtual network design best practices" })

// Get NSG guidance
microsoft_docs_search({ query: "network security group rules best practices Azure" })

// Find private endpoint documentation
microsoft_docs_search({ query: "private endpoint configuration DNS Azure" })

// Get hub-spoke architecture guidance
microsoft_docs_search({ query: "hub-spoke network topology Azure architecture" })
```

## Hub-Spoke Topology

### Overview

Hub-spoke is the recommended topology for enterprise Azure deployments. It provides centralized connectivity, security, and shared services.

```
                    ┌─────────────────────────────────────┐
                    │            Hub VNet                 │
                    │  ┌─────────┐  ┌──────────────────┐  │
                    │  │ Firewall│  │ VPN/ExpressRoute │  │
                    │  └─────────┘  │    Gateway       │  │
                    │               └──────────────────┘  │
                    │  ┌──────────────────────────────┐   │
                    │  │     Shared Services          │   │
                    │  │  (DNS, Bastion, etc.)        │   │
                    │  └──────────────────────────────┘   │
                    └──────────────┬──────────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
    │   Spoke VNet    │  │   Spoke VNet    │  │   Spoke VNet    │
    │   (Workload A)  │  │   (Workload B)  │  │   (Workload C)  │
    │                 │  │                 │  │                 │
    │ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │
    │ │ App Subnet  │ │  │ │ App Subnet  │ │  │ │ App Subnet  │ │
    │ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │
    │ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │
    │ │ Data Subnet │ │  │ │ Data Subnet │ │  │ │ Data Subnet │ │
    │ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │
    └─────────────────┘  └─────────────────┘  └─────────────────┘
```

### Hub VNet Design

```javascript
// Search for hub-spoke architecture guidance
microsoft_docs_search({
  "query": "hub-spoke network topology Azure reference architecture"
})

// Get Azure Firewall documentation
microsoft_docs_search({
  "query": "Azure Firewall configuration best practices"
})
```

**Hub Subnet Recommendations:**

| Subnet | Purpose | Recommended Size |
|--------|---------|------------------|
| GatewaySubnet | VPN/ExpressRoute Gateway | /27 minimum |
| AzureFirewallSubnet | Azure Firewall | /26 (required) |
| AzureBastionSubnet | Azure Bastion | /26 minimum |
| SharedServicesSubnet | DNS, AD, etc. | /24 |

### Spoke VNet Design

**Spoke Subnet Recommendations:**

| Subnet | Purpose | Recommended Size |
|--------|---------|------------------|
| AppSubnet | Application tier | /24 |
| DataSubnet | Database tier | /24 |
| PrivateEndpointSubnet | Private endpoints | /24 |
| IntegrationSubnet | App Service VNet Integration | /26 minimum |

### IP Address Planning

**Best Practices:**
1. **Use RFC 1918 ranges** - 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
2. **Plan for growth** - Allocate larger ranges than immediately needed
3. **Avoid overlaps** - Ensure no overlap with on-premises or other VNets
4. **Document allocations** - Maintain an IP address management (IPAM) system

**Example Allocation:**

| VNet | Address Space | Purpose |
|------|---------------|---------|
| Hub | 10.0.0.0/16 | Shared services |
| Spoke-Dev | 10.1.0.0/16 | Development |
| Spoke-Staging | 10.2.0.0/16 | Staging |
| Spoke-Prod | 10.3.0.0/16 | Production |

## Network Security Groups (NSGs)

### NSG Design Principles

1. **Apply at subnet level** - Easier to manage than NIC-level
2. **Use service tags** - Leverage Azure-managed IP ranges
3. **Deny by default** - Only allow required traffic
4. **Log all traffic** - Enable NSG flow logs for troubleshooting
5. **Use Application Security Groups** - Group VMs by function

### Common NSG Rule Patterns

#### Web Application Tier

```javascript
// Search for NSG best practices
microsoft_docs_search({
  "query": "NSG rules web application tier Azure"
})

// Get service tags documentation
microsoft_docs_search({
  "query": "Azure service tags network security groups"
})
```

| Priority | Name | Direction | Source | Destination | Port | Action |
|----------|------|-----------|--------|-------------|------|--------|
| 100 | AllowHTTPS | Inbound | Internet | AppSubnet | 443 | Allow |
| 110 | AllowHTTP | Inbound | Internet | AppSubnet | 80 | Allow |
| 120 | AllowAzureLB | Inbound | AzureLoadBalancer | AppSubnet | * | Allow |
| 4096 | DenyAllInbound | Inbound | * | * | * | Deny |

#### Database Tier

| Priority | Name | Direction | Source | Destination | Port | Action |
|----------|------|-----------|--------|-------------|------|--------|
| 100 | AllowAppToSQL | Inbound | AppSubnet | DataSubnet | 1433 | Allow |
| 110 | AllowAppToPostgres | Inbound | AppSubnet | DataSubnet | 5432 | Allow |
| 120 | AllowBastionRDP | Inbound | AzureBastionSubnet | DataSubnet | 3389 | Allow |
| 4096 | DenyAllInbound | Inbound | * | * | * | Deny |

#### Private Endpoint Subnet

| Priority | Name | Direction | Source | Destination | Port | Action |
|----------|------|-----------|--------|-------------|------|--------|
| 100 | AllowVNetInbound | Inbound | VirtualNetwork | PrivateEndpointSubnet | * | Allow |
| 4096 | DenyAllInbound | Inbound | * | * | * | Deny |

### Service Tags Reference

| Service Tag | Description |
|-------------|-------------|
| `Internet` | Public internet |
| `VirtualNetwork` | VNet address space + peered VNets |
| `AzureLoadBalancer` | Azure Load Balancer health probes |
| `AzureCloud` | All Azure public IPs |
| `AzureCloud.EastUS` | Azure IPs in specific region |
| `Storage` | Azure Storage |
| `Sql` | Azure SQL Database |
| `AzureKeyVault` | Azure Key Vault |
| `AzureContainerRegistry` | Azure Container Registry |

### Application Security Groups (ASGs)

Use ASGs to group VMs by function rather than IP address:

```bicep
// Define ASGs
resource webServersAsg 'Microsoft.Network/applicationSecurityGroups@2023-05-01' = {
  name: 'asg-web-servers'
  location: location
}

resource dbServersAsg 'Microsoft.Network/applicationSecurityGroups@2023-05-01' = {
  name: 'asg-db-servers'
  location: location
}

// Use in NSG rules
resource nsg 'Microsoft.Network/networkSecurityGroups@2023-05-01' = {
  name: 'nsg-app'
  location: location
  properties: {
    securityRules: [
      {
        name: 'AllowWebToDb'
        properties: {
          priority: 100
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceApplicationSecurityGroups: [{ id: webServersAsg.id }]
          destinationApplicationSecurityGroups: [{ id: dbServersAsg.id }]
          sourcePortRange: '*'
          destinationPortRange: '1433'
        }
      }
    ]
  }
}
```

## Private Endpoints

### When to Use Private Endpoints

Use private endpoints when you need to:
- Access PaaS services over private IP
- Meet compliance requirements for private connectivity
- Eliminate data exfiltration risks
- Integrate with on-premises networks

### Private Endpoint Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         VNet                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  App Subnet                               │  │
│  │  ┌─────────────┐                                          │  │
│  │  │   App VM    │ ────────────────────────────────────┐    │  │
│  │  └─────────────┘                                     │    │  │
│  └──────────────────────────────────────────────────────│────┘  │
│  ┌──────────────────────────────────────────────────────│────┐  │
│  │              Private Endpoint Subnet                 │    │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │    │  │
│  │  │ PE: Storage │  │ PE: SQL DB  │  │ PE: Key Vault│  │    │  │
│  │  │ 10.0.2.4    │  │ 10.0.2.5    │  │ 10.0.2.6     │◀─┘    │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘       │  │
│  └─────────│────────────────│────────────────│───────────────┘  │
└────────────│────────────────│────────────────│──────────────────┘
             │                │                │
             ▼                ▼                ▼
    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │   Storage   │  │   SQL DB    │  │  Key Vault  │
    │   Account   │  │             │  │             │
    └─────────────┘  └─────────────┘  └─────────────┘
```

### Private Endpoint Configuration

```javascript
// Search for private endpoint documentation
microsoft_docs_search({
  "query": "private endpoint configuration Bicep Azure"
})

// Get private DNS zone guidance
microsoft_docs_search({
  "query": "private DNS zone private endpoint Azure"
})

// Find code samples
microsoft_code_sample_search({
  "query": "private endpoint Bicep template"
})
```

### Private DNS Zones

Each Azure service requires a specific private DNS zone:

| Service | Private DNS Zone |
|---------|------------------|
| Storage (blob) | `privatelink.blob.core.windows.net` |
| Storage (file) | `privatelink.file.core.windows.net` |
| SQL Database | `privatelink.database.windows.net` |
| Cosmos DB | `privatelink.documents.azure.com` |
| Key Vault | `privatelink.vaultcore.azure.net` |
| Azure Container Registry | `privatelink.azurecr.io` |
| App Service | `privatelink.azurewebsites.net` |
| Azure Functions | `privatelink.azurewebsites.net` |
| Event Hubs | `privatelink.servicebus.windows.net` |
| Service Bus | `privatelink.servicebus.windows.net` |

### Private Endpoint Best Practices

1. **Centralize DNS zones** - Host private DNS zones in the hub VNet
2. **Link to all VNets** - Ensure DNS zones are linked to all spoke VNets
3. **Disable public access** - After creating private endpoints, disable public network access
4. **Use dedicated subnet** - Create a subnet specifically for private endpoints
5. **Plan IP addresses** - Each private endpoint consumes one IP address

## Common Network Patterns

### Pattern 1: Web Application with Private Backend

```javascript
// Search for App Service VNet integration
microsoft_docs_search({
  "query": "App Service VNet integration private endpoint Azure"
})

// Get architecture guidance
microsoft_docs_search({
  "query": "web application private backend Azure architecture"
})
```

```
┌────────────────────────────────────────────────────────────────┐
│                         VNet                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              App Service Integration Subnet             │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │              App Service                        │    │   │
│  │  │         (VNet Integration enabled)              │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Private Endpoint Subnet                    │   │
│  │  ┌─────────────┐  ┌───────────────┐                     │   │
│  │  │ PE: SQL DB  │  │ PE: Key Vault │                     │   │
│  │  └─────────────┘  └───────────────┘                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

### Pattern 2: AKS with Private Cluster

```javascript
// Search for AKS private cluster documentation
microsoft_docs_search({
  "query": "AKS private cluster networking configuration"
})

// Get AKS networking best practices
microsoft_docs_search({
  "query": "AKS networking CNI vs kubenet Azure"
})
```

```
┌─────────────────────────────────────────────────────────────────┐
│                         VNet                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    AKS Subnet (/22)                      │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │              AKS Cluster (Private)               │    │   │
│  │  │         API Server: Private Endpoint             │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Private Endpoint Subnet                     │   │
│  │  ┌─────────────┐  ┌───────────────┐  ┌─────────────┐     │   │
│  │  │ PE: ACR     │  │ PE: Key Vault │  │ PE: Storage │     │   │
│  │  └─────────────┘  └───────────────┘  └─────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 3: Multi-Region with Traffic Manager

```
                    ┌─────────────────────┐
                    │   Traffic Manager   │
                    │   (Global LB)       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐               ┌─────────────────┐
    │   East US       │               │   West US       │
    │   Region        │               │   Region        │
    │                 │               │                 │
    │ ┌─────────────┐ │               │ ┌─────────────┐ │
    │ │ App Gateway │ │               │ │ App Gateway │ │
    │ └──────┬──────┘ │               │ └──────┬──────┘ │
    │        │        │               │        │        │
    │ ┌──────▼──────┐ │               │ ┌──────▼──────┐ │
    │ │   VNet      │ │               │ │   VNet      │ │
    │ │ (Hub-Spoke) │ │               │ │ (Hub-Spoke) │ │
    │ └─────────────┘ │               │ └─────────────┘ │
    └─────────────────┘               └─────────────────┘
```

## Network Security Checklist

Use this checklist when designing network architecture:

- [ ] **IP Address Planning**
  - [ ] Non-overlapping address spaces
  - [ ] Room for growth
  - [ ] Documented allocations

- [ ] **Network Segmentation**
  - [ ] Separate subnets for different tiers
  - [ ] NSGs applied to all subnets
  - [ ] Service endpoints or private endpoints for PaaS

- [ ] **Access Control**
  - [ ] Deny-by-default NSG rules
  - [ ] Service tags used where possible
  - [ ] ASGs for VM grouping

- [ ] **Private Connectivity**
  - [ ] Private endpoints for sensitive services
  - [ ] Private DNS zones configured
  - [ ] Public access disabled where possible

- [ ] **Monitoring**
  - [ ] NSG flow logs enabled
  - [ ] Network Watcher configured
  - [ ] Azure Firewall logs (if applicable)

## Troubleshooting Network Issues

```javascript
// Search for network troubleshooting guidance
microsoft_docs_search({
  "query": "Azure network connectivity troubleshooting"
})

// Get NSG diagnostics documentation
microsoft_docs_search({
  "query": "NSG flow logs troubleshooting Azure"
})

// Find Network Watcher guidance
microsoft_docs_search({
  "query": "Network Watcher connection troubleshoot Azure"
})
```

### Common Issues

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Cannot reach private endpoint | DNS not resolving | Check private DNS zone links |
| NSG blocking traffic | Missing allow rule | Review NSG rules, check effective rules |
| VNet peering not working | Peering not established both ways | Create peering in both directions |
| Cannot reach on-premises | Route table missing | Add UDR for on-premises ranges |

## Related Steering Files

- **[iac-workflows.md](./iac-workflows.md)** - IaC tool selection, Bicep/Terraform patterns

## Resources

- [Azure Virtual Network Documentation](https://docs.microsoft.com/azure/virtual-network/)
- [Hub-Spoke Network Topology](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
- [Private Endpoint Documentation](https://docs.microsoft.com/azure/private-link/private-endpoint-overview)
- [NSG Documentation](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [Azure Network Security Best Practices](https://docs.microsoft.com/azure/security/fundamentals/network-best-practices)

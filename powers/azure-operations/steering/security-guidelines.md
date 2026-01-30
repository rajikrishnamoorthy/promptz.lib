# Security Guidelines

This steering file provides guidance for Azure RBAC patterns, policy compliance, security best practices, and least privilege access using the azure-operations power.

## Security Assessment Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                     Security Assessment Flow                    │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
              ┌──────────────────────────────────────┐
              │        What security aspect?         │
              └──────────────────────────────────────┘
                     │              │              │
                  Access        Compliance      Secrets
                     │              │              │
                     ▼              ▼              ▼
         ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
         │ role_assignment │ │ policy_         │ │ keyvault_       │
         │ _list           │ │ assignment_list │ │ secret_list     │
         │                 │ │                 │ │ keyvault_       │
         │ advisor_        │ │ advisor_        │ │ key_list        │
         │ recommendation  │ │ recommendation  │ │                 │
         │ _list (Security)│ │ _list           │ │                 │
         └─────────────────┘ └─────────────────┘ └─────────────────┘
```

## RBAC (Role-Based Access Control)

### RBAC Fundamentals

Azure RBAC uses three key concepts:

```
┌──────────────────────────────────────────────────────────────────┐
│                    RBAC Assignment                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   WHO (Security Principal)                                       │
│   ├── User                                                       │
│   ├── Group                                                      │
│   ├── Service Principal                                          │
│   └── Managed Identity                                           │
│                                                                  │
│   WHAT (Role Definition)                                         │
│   ├── Owner                                                      │
│   ├── Contributor                                                │
│   ├── Reader                                                     │
│   └── Custom Roles                                               │
│                                                                  │
│   WHERE (Scope)                                                  │
│   ├── Management Group                                           │
│   ├── Subscription                                               │
│   ├── Resource Group                                             │
│   └── Resource                                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Common Built-in Roles

| Role | Description | Use Case |
|------|-------------|----------|
| **Owner** | Full access + can assign roles | Subscription admins only |
| **Contributor** | Full access, cannot assign roles | DevOps teams |
| **Reader** | Read-only access | Auditors, monitoring |
| **User Access Administrator** | Manage user access | Security team |
| **Storage Blob Data Contributor** | Read/write blob data | Applications |
| **Key Vault Secrets User** | Read secrets | Applications |
| **AKS Cluster Admin** | Full AKS access | Kubernetes admins |
| **SQL DB Contributor** | Manage SQL databases | Database admins |

### Auditing RBAC Assignments

```javascript
// Step 1: List all role assignments at subscription level
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
})

// Step 2: List role assignments for a specific resource group
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-myapp-prod"
})

// Step 3: List role assignments for a specific user/service principal
role_assignment_list({
  "assignee": "user@example.com"
})

// Step 4: Get security recommendations
advisor_recommendation_list({
  "category": "Security"
})
```

### Least Privilege Patterns

#### Pattern 1: Application Access to Storage

**Instead of:** Contributor role on storage account

**Use:** Storage Blob Data Contributor on specific container

```javascript
// Audit current storage access
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-data-prod/providers/Microsoft.Storage/storageAccounts/stmyappprod001"
})
```

#### Pattern 2: Application Access to Key Vault

**Instead of:** Key Vault Contributor

**Use:** Key Vault Secrets User (read-only) or Key Vault Secrets Officer (read/write)

```javascript
// Audit Key Vault access
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-security-prod/providers/Microsoft.KeyVault/vaults/kv-myapp-prod"
})
```

#### Pattern 3: CI/CD Pipeline Access

**Instead of:** Contributor on subscription

**Use:** Contributor on specific resource groups + specific data plane roles

```javascript
// Audit service principal access
role_assignment_list({
  "assignee": "00000000-0000-0000-0000-000000000000"  // Service principal object ID
})
```

### RBAC Best Practices Checklist

- [ ] **Use groups over users** - Assign roles to Azure AD groups, not individuals
- [ ] **Scope narrowly** - Assign at resource group or resource level, not subscription
- [ ] **Prefer built-in roles** - Use custom roles only when necessary
- [ ] **Use managed identities** - Avoid service principals with secrets
- [ ] **Regular access reviews** - Audit assignments quarterly
- [ ] **Remove Owner assignments** - Minimize Owner role usage
- [ ] **Document exceptions** - Track any elevated access with justification

## Azure Policy Compliance

### Understanding Azure Policy

Azure Policy enforces organizational standards and assesses compliance at scale.

```
┌──────────────────────────────────────────────────────────────────┐
│                    Policy Hierarchy                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Management Group (Inherited by all subscriptions)              │
│   └── Subscription (Inherited by all resource groups)            │
│       └── Resource Group (Inherited by all resources)            │
│           └── Resource                                           │
│                                                                  │
│   Policy Effects:                                                │
│   • Deny - Block non-compliant resources                         │
│   • Audit - Log non-compliance                                   │
│   • Modify - Auto-remediate resources                            │
│   • DeployIfNotExists - Deploy missing resources                 │
│   • AuditIfNotExists - Audit missing resources                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Auditing Policy Compliance

```javascript
// Step 1: List all policy assignments
policy_assignment_list({
  "subscription": "my-subscription"
})

// Step 2: List policy assignments at a specific scope
policy_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-myapp-prod"
})
```

### Common Security Policies

| Policy | Effect | Purpose |
|--------|--------|---------|
| Require HTTPS for storage | Deny | Enforce encrypted connections |
| Require encryption at rest | Deny | Protect data at rest |
| Allowed locations | Deny | Data residency compliance |
| Require tags | Deny | Cost allocation and governance |
| Audit VMs without managed disks | Audit | Identify legacy configurations |
| Deploy diagnostic settings | DeployIfNotExists | Ensure logging |
| Require private endpoints | Deny | Network isolation |

### Policy Compliance Workflow

```javascript
// Step 1: List all policy assignments to understand what's enforced
policy_assignment_list({
  "subscription": "my-subscription"
})

// Step 2: Check Advisor for security recommendations
advisor_recommendation_list({
  "category": "Security"
})

// Step 3: Review specific resource compliance
// (Check if resources meet policy requirements)
storage_account_list({
  "resource-group": "rg-myapp-prod"
})
```

## Secrets Management Security

### Key Vault Security Best Practices

```
┌──────────────────────────────────────────────────────────────────┐
│                 Key Vault Security Layers                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Network Security                                               │
│   ├── Private endpoints (recommended)                            │
│   ├── Service endpoints                                          │
│   └── Firewall rules                                             │
│                                                                  │
│   Access Control                                                 │
│   ├── Azure RBAC (recommended)                                   │
│   └── Access policies (legacy)                                   │
│                                                                  │
│   Data Protection                                                │
│   ├── Soft delete (enabled by default)                           │
│   ├── Purge protection                                           │
│   └── HSM-backed keys (Premium SKU)                              │
│                                                                  │
│   Monitoring                                                     │
│   ├── Diagnostic logs                                            │
│   └── Azure Monitor alerts                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Key Vault Security Audit

```javascript
// Step 1: List all Key Vaults
keyvault_list({
  "subscription": "my-subscription"
})

// Step 2: Audit secrets in each vault
keyvault_secret_list({
  "vault-name": "kv-myapp-prod-001"
})

// Step 3: Audit cryptographic keys
keyvault_key_list({
  "vault-name": "kv-myapp-prod-001"
})

// Step 4: Check RBAC assignments on Key Vault
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-security-prod/providers/Microsoft.KeyVault/vaults/kv-myapp-prod-001"
})
```

### Secrets Management Checklist

- [ ] **Enable soft delete** - Protect against accidental deletion
- [ ] **Enable purge protection** - Prevent permanent deletion
- [ ] **Use private endpoints** - Eliminate public network access
- [ ] **Use Azure RBAC** - Prefer RBAC over access policies
- [ ] **Rotate secrets regularly** - Implement rotation policies
- [ ] **Use managed identities** - Avoid storing credentials
- [ ] **Enable diagnostic logging** - Track all access
- [ ] **Set expiration dates** - Enforce secret rotation

## Storage Security

### Storage Security Layers

```javascript
// Step 1: List storage accounts to audit
storage_account_list({
  "resource-group": "rg-data-prod"
})

// Step 2: Check RBAC assignments
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-data-prod/providers/Microsoft.Storage/storageAccounts/stmyappprod001"
})
```

### Storage Security Checklist

- [ ] **Require secure transfer** - HTTPS only
- [ ] **Disable public blob access** - Unless explicitly needed
- [ ] **Use private endpoints** - For production workloads
- [ ] **Enable storage firewall** - Restrict network access
- [ ] **Use Azure AD authentication** - Prefer over shared keys
- [ ] **Enable soft delete** - For blobs and containers
- [ ] **Enable versioning** - Track changes
- [ ] **Enable encryption** - Customer-managed keys for sensitive data

## Database Security

### Database Security Patterns

| Database | Security Features |
|----------|-------------------|
| **Cosmos DB** | Azure AD auth, RBAC, encryption, private endpoints |
| **Azure SQL** | Azure AD auth, TDE, Always Encrypted, private endpoints |
| **PostgreSQL** | Azure AD auth, SSL enforcement, private endpoints |
| **MySQL** | Azure AD auth, SSL enforcement, private endpoints |
| **Redis** | Access keys, private endpoints, TLS |

### Database Security Audit

```javascript
// Step 1: List all database resources
cosmos_account_list({ "subscription": "my-subscription" })
sql_server_list({ "subscription": "my-subscription" })
postgres_server_list({ "subscription": "my-subscription" })
mysql_server_list({ "subscription": "my-subscription" })
redis_list({ "subscription": "my-subscription" })

// Step 2: Check RBAC on database resources
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-data-prod/providers/Microsoft.DocumentDB/databaseAccounts/cosmos-myapp-prod"
})

// Step 3: Review security recommendations
advisor_recommendation_list({
  "category": "Security"
})
```

## Container Security

### AKS Security Best Practices

```javascript
// Step 1: List AKS clusters
aks_cluster_list({
  "subscription": "my-subscription"
})

// Step 2: Review node pools
aks_nodepool_list({
  "cluster-name": "aks-myapp-prod",
  "resource-group": "rg-k8s-prod"
})

// Step 3: Check RBAC assignments
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-k8s-prod/providers/Microsoft.ContainerService/managedClusters/aks-myapp-prod"
})
```

### AKS Security Checklist

- [ ] **Enable Azure AD integration** - Use Azure AD for authentication
- [ ] **Use Azure RBAC for Kubernetes** - Unified access control
- [ ] **Enable private cluster** - No public API server
- [ ] **Use managed identities** - For cluster and workload identity
- [ ] **Enable Azure Policy for AKS** - Enforce pod security
- [ ] **Scan container images** - Enable Microsoft Defender
- [ ] **Use private ACR** - Private endpoints for registry
- [ ] **Network policies** - Restrict pod-to-pod traffic

### Container Registry Security

```javascript
// Step 1: List container registries
acr_list({
  "subscription": "my-subscription"
})

// Step 2: Check RBAC assignments
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-containers-prod/providers/Microsoft.ContainerRegistry/registries/acrmyappprod"
})
```

## Security Audit Workflow

### Comprehensive Security Review

```javascript
// Step 1: Get all security recommendations
advisor_recommendation_list({
  "category": "Security"
})

// Step 2: Audit RBAC at subscription level
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
})

// Step 3: Review policy compliance
policy_assignment_list({
  "subscription": "my-subscription"
})

// Step 4: Audit Key Vaults
keyvault_list({ "subscription": "my-subscription" })

// Step 5: Check for over-privileged access
// Look for Owner/Contributor at subscription level
// Look for broad data plane access
```

### Security Findings Response Matrix

| Finding | Severity | Action |
|---------|----------|--------|
| Owner role at subscription | High | Review and remove if not needed |
| Contributor on subscription | Medium | Scope to resource groups |
| Public storage access | High | Disable or use SAS tokens |
| Missing private endpoints | Medium | Implement for production |
| Expired secrets | High | Rotate immediately |
| Missing encryption | High | Enable encryption at rest |
| No diagnostic logging | Medium | Enable diagnostic settings |
| Missing MFA | Critical | Enforce conditional access |

## Compliance Frameworks

### Common Compliance Requirements

| Framework | Key Azure Controls |
|-----------|-------------------|
| **SOC 2** | Access control, encryption, logging |
| **HIPAA** | Encryption, access control, audit logs |
| **PCI DSS** | Network segmentation, encryption, access control |
| **GDPR** | Data residency, encryption, access control |
| **ISO 27001** | Comprehensive security controls |

### Compliance Audit Steps

```javascript
// Step 1: Review policy assignments for compliance policies
policy_assignment_list({
  "subscription": "my-subscription"
})

// Step 2: Check encryption status
// Review storage accounts, databases, Key Vaults

// Step 3: Audit access control
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
})

// Step 4: Review Advisor recommendations
advisor_recommendation_list({
  "category": "Security"
})
```

## Security Incident Response

### Investigation Workflow

```javascript
// Step 1: Identify affected resources
storage_account_list({ "resource-group": "affected-rg" })
keyvault_list({ "resource-group": "affected-rg" })

// Step 2: Audit access to affected resources
role_assignment_list({
  "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/affected-rg"
})

// Step 3: Check for suspicious role assignments
role_assignment_list({
  "assignee": "suspicious-user@example.com"
})

// Step 4: Review Key Vault access
keyvault_secret_list({
  "vault-name": "affected-keyvault"
})
```

## Related Steering Files

- **[operations-best-practices.md](./operations-best-practices.md)** - Resource management patterns, naming conventions

## Resources

- [Azure RBAC Documentation](https://docs.microsoft.com/azure/role-based-access-control/)
- [Azure Policy Documentation](https://docs.microsoft.com/azure/governance/policy/)
- [Azure Security Best Practices](https://docs.microsoft.com/azure/security/fundamentals/best-practices-and-patterns)
- [Azure Key Vault Security](https://docs.microsoft.com/azure/key-vault/general/security-features)
- [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/)
- [Microsoft Defender for Cloud](https://docs.microsoft.com/azure/defender-for-cloud/)

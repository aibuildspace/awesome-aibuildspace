---
name: bicep
description: |
  Azure Bicep IaC expert for Azure resource deployments.
  Generates modular Bicep templates, parameter files, deployment stacks,
  and CI/CD integration. Covers networking, compute, data, identity, and monitoring.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[task: module|deploy|migrate|refactor|review] [resource or architecture description]"
---

## Role

You are a Bicep specialist. You write Azure infrastructure code that is modular, type-safe, and follows Azure Verified Modules patterns.

## Task Router

| Task | What You Do |
|------|------------|
| `module` | Generate a reusable Bicep module for a described resource/pattern |
| `deploy` | Create a full deployment (main.bicep + modules + params) |
| `migrate` | Convert ARM JSON templates to Bicep |
| `refactor` | Restructure existing Bicep for modularity, DRY, best practices |
| `review` | Review existing Bicep code for correctness, security, patterns |

---

## Project Structure

### Standard Layout

```
infra/
├── main.bicep                  # Orchestrator — calls modules
├── main.bicepparam             # Default parameter values
├── bicepconfig.json            # Linter rules, module aliases
├── environments/
│   ├── dev.bicepparam
│   ├── staging.bicepparam
│   └── prod.bicepparam
├── modules/
│   ├── networking/
│   │   ├── vnet.bicep
│   │   ├── nsg.bicep
│   │   └── private-endpoint.bicep
│   ├── compute/
│   │   ├── app-service.bicep
│   │   ├── function-app.bicep
│   │   └── aks.bicep
│   ├── data/
│   │   ├── sql-server.bicep
│   │   ├── cosmos-db.bicep
│   │   └── storage-account.bicep
│   ├── identity/
│   │   └── managed-identity.bicep
│   └── monitoring/
│       ├── log-analytics.bicep
│       └── app-insights.bicep
└── scripts/
    ├── deploy.sh
    └── what-if.sh
```

### bicepconfig.json

```json
{
  "analyzers": {
    "core": {
      "rules": {
        "no-hardcoded-env-urls": { "level": "error" },
        "no-unused-params": { "level": "error" },
        "no-unused-vars": { "level": "error" },
        "prefer-interpolation": { "level": "warning" },
        "secure-parameter-default": { "level": "error" },
        "simplify-interpolation": { "level": "warning" },
        "use-resource-id-functions": { "level": "warning" },
        "use-stable-resource-identifiers": { "level": "warning" }
      }
    }
  },
  "moduleAliases": {
    "br": {
      "avm": {
        "registry": "mcr.microsoft.com",
        "modulePath": "bicep/avm"
      }
    }
  }
}
```

---

## Module Template

```bicep
// modules/compute/app-service.bicep

metadata name = 'App Service'
metadata description = 'Deploys an App Service with managed identity and diagnostics'
metadata version = '1.0.0'

// ── Parameters ───────────────────────────────

@description('Resource name')
@minLength(3)
@maxLength(60)
param name string

@description('Azure region')
param location string

@description('Environment')
@allowed(['dev', 'staging', 'prod'])
param environment string

@description('App Service Plan resource ID')
param appServicePlanId string

@description('Application settings')
param appSettings object = {}

@description('Tags applied to all resources')
param tags object = {}

@description('Enable VNet integration')
param enableVnetIntegration bool = false

@description('Subnet ID for VNet integration')
param subnetId string = ''

// ── Variables ────────────────────────────────

var defaultAppSettings = {
  WEBSITE_RUN_FROM_PACKAGE: '1'
  APPLICATIONINSIGHTS_CONNECTION_STRING: appInsights.properties.ConnectionString
}

var mergedAppSettings = union(defaultAppSettings, appSettings)

// ── Resources ────────────────────────────────

resource appService 'Microsoft.Web/sites@2023-12-01' = {
  name: name
  location: location
  tags: tags
  kind: 'app,linux'
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: appServicePlanId
    httpsOnly: true
    siteConfig: {
      linuxFxVersion: 'PYTHON|3.12'
      minTlsVersion: '1.2'
      ftpsState: 'Disabled'
      http20Enabled: true
      alwaysOn: environment == 'prod'
      appSettings: [
        for (setting, _) in items(mergedAppSettings): {
          name: setting.key
          value: setting.value
        }
      ]
    }
    virtualNetworkSubnetId: enableVnetIntegration ? subnetId : null
  }
}

resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: 'appi-${name}'
  location: location
  tags: tags
  kind: 'web'
  properties: {
    Application_Type: 'web'
    WorkspaceResourceId: logAnalyticsWorkspaceId
  }
}

// Diagnostic settings
resource diagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'diag-${name}'
  scope: appService
  properties: {
    workspaceId: logAnalyticsWorkspaceId
    logs: [
      { categoryGroup: 'allLogs', enabled: true }
    ]
    metrics: [
      { category: 'AllMetrics', enabled: true }
    ]
  }
}

// ── Outputs ──────────────────────────────────

@description('App Service resource ID')
output id string = appService.id

@description('App Service default hostname')
output defaultHostname string = appService.properties.defaultHostName

@description('App Service managed identity principal ID')
output principalId string = appService.identity.principalId

@description('App Service name')
output name string = appService.name
```

---

## Orchestrator Pattern

### main.bicep

```bicep
targetScope = 'resourceGroup'

// ── Parameters ───────────────────────────────

@description('Environment')
@allowed(['dev', 'staging', 'prod'])
param environment string

@description('Azure region')
param location string = resourceGroup().location

@description('Project name')
@minLength(3)
@maxLength(20)
param projectName string

@secure()
@description('SQL admin password')
param sqlAdminPassword string

// ── Variables ────────────────────────────────

var namePrefix = '${projectName}-${environment}'
var commonTags = {
  Environment: environment
  Project: projectName
  ManagedBy: 'bicep'
  DeployedAt: utcNow('yyyy-MM-dd')
}

// ── Modules ──────────────────────────────────

module networking 'modules/networking/vnet.bicep' = {
  name: 'networking-${uniqueString(deployment().name)}'
  params: {
    name: 'vnet-${namePrefix}'
    location: location
    environment: environment
    tags: commonTags
  }
}

module database 'modules/data/sql-server.bicep' = {
  name: 'database-${uniqueString(deployment().name)}'
  params: {
    name: 'sql-${namePrefix}'
    location: location
    environment: environment
    adminPassword: sqlAdminPassword
    subnetId: networking.outputs.dataSubnetId
    tags: commonTags
  }
}

module app 'modules/compute/app-service.bicep' = {
  name: 'app-${uniqueString(deployment().name)}'
  params: {
    name: 'app-${namePrefix}'
    location: location
    environment: environment
    appServicePlanId: appServicePlan.outputs.id
    enableVnetIntegration: true
    subnetId: networking.outputs.appSubnetId
    appSettings: {
      DATABASE_URL: database.outputs.connectionString
    }
    tags: commonTags
  }
}

// ── Outputs ──────────────────────────────────

output appUrl string = 'https://${app.outputs.defaultHostname}'
output sqlServerFqdn string = database.outputs.fullyQualifiedDomainName
```

### Parameter File (.bicepparam)

```bicep
// environments/prod.bicepparam
using '../main.bicep'

param environment = 'prod'
param projectName = 'myapp'
param location = 'australiaeast'
param sqlAdminPassword = readEnvironmentVariable('SQL_ADMIN_PASSWORD')
```

---

## Azure Verified Modules (AVM)

Use AVM from the Bicep public registry when available:

```bicep
// Use AVM instead of writing from scratch
module vnet 'br/public:avm/res/network/virtual-network:0.5.0' = {
  name: 'vnet-deployment'
  params: {
    name: 'vnet-${namePrefix}'
    location: location
    addressPrefixes: ['10.0.0.0/16']
    subnets: [
      { name: 'snet-app', addressPrefix: '10.0.1.0/24' }
      { name: 'snet-data', addressPrefix: '10.0.2.0/24' }
    ]
    tags: commonTags
  }
}

module keyVault 'br/public:avm/res/key-vault/vault:0.9.0' = {
  name: 'kv-deployment'
  params: {
    name: 'kv-${namePrefix}'
    location: location
    enableRbacAuthorization: true
    enableSoftDelete: true
    enablePurgeProtection: true
    tags: commonTags
  }
}
```

### When to Use AVM vs Custom Modules

| Use AVM | Use Custom |
|---------|-----------|
| Standard resource with common config | Highly opinionated pattern specific to your org |
| Want automatic updates from Microsoft | Need tight control over API versions |
| Rapid prototyping | Complex multi-resource compositions |
| Compliance with enterprise standards | Unique naming/tagging requirements |

---

## Deployment Stacks

```bicep
// Deployment stacks (preview) for lifecycle management
// Ensures resources not in template are cleaned up

// Deploy via CLI:
// az stack group create \
//   --name myapp-prod \
//   --resource-group rg-myapp-prod \
//   --template-file main.bicep \
//   --parameters environments/prod.bicepparam \
//   --deny-settings-mode denyWriteAndDelete \
//   --action-on-unmanage deleteResources
```

### Deny Settings

| Mode | Effect |
|------|--------|
| `none` | No restrictions |
| `denyDelete` | Prevent deletion of managed resources |
| `denyWriteAndDelete` | Prevent modification and deletion (prod) |

---

## Key Language Features

### Conditional Deployment

```bicep
// Deploy resource only in production
resource waf 'Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies@2023-11-01' = if (environment == 'prod') {
  name: 'waf-${namePrefix}'
  location: location
  properties: { ... }
}
```

### Loops

```bicep
// Array loop
param privateEndpoints array = [
  { name: 'sql', groupId: 'sqlServer', resourceId: sqlServerId }
  { name: 'kv', groupId: 'vault', resourceId: keyVaultId }
  { name: 'st', groupId: 'blob', resourceId: storageId }
]

resource pe 'Microsoft.Network/privateEndpoints@2023-11-01' = [
  for endpoint in privateEndpoints: {
    name: 'pe-${endpoint.name}-${namePrefix}'
    location: location
    properties: {
      subnet: { id: dataSubnetId }
      privateLinkServiceConnections: [
        {
          name: 'plsc-${endpoint.name}'
          properties: {
            privateLinkServiceId: endpoint.resourceId
            groupIds: [endpoint.groupId]
          }
        }
      ]
    }
  }
]
```

### User-Defined Types

```bicep
@export()
type subnetConfig = {
  @description('Subnet name')
  name: string

  @description('CIDR prefix')
  addressPrefix: string

  @description('NSG resource ID')
  nsgId: string?

  @description('Delegation')
  delegation: string?
}

param subnets subnetConfig[]
```

### User-Defined Functions

```bicep
@export()
func generateResourceName(prefix string, project string, env string, suffix string) string =>
  '${prefix}-${project}-${env}-${suffix}'

// Usage
var appName = generateResourceName('app', projectName, environment, 'api')
```

---

## ARM → Bicep Migration

```bash
# Decompile existing ARM JSON to Bicep
az bicep decompile --file azuredeploy.json

# Paste ARM JSON and convert interactively
az bicep decompile --file template.json --force

# Export existing resource group to Bicep
az group export --name my-rg --include-parameter-default-value > exported.json
az bicep decompile --file exported.json
```

### Post-Migration Cleanup
1. Fix all linter warnings (`bicepconfig.json` rules)
2. Replace `any()` casts with proper types
3. Extract repeated resources into modules
4. Add descriptions to all parameters and outputs
5. Replace hardcoded values with parameters
6. Add validation decorators (`@minLength`, `@allowed`, etc.)

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main]
    paths: ['infra/**']

permissions:
  id-token: write   # OIDC
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Bicep Build
        run: az bicep build --file infra/main.bicep
      - name: What-If
        run: |
          az deployment group what-if \
            --resource-group rg-myapp-prod \
            --template-file infra/main.bicep \
            --parameters infra/environments/prod.bicepparam

  deploy:
    needs: validate
    runs-on: ubuntu-latest
    environment: production   # Requires approval
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Deploy
        run: |
          az deployment group create \
            --resource-group rg-myapp-prod \
            --template-file infra/main.bicep \
            --parameters infra/environments/prod.bicepparam
```

### What-If Before Apply

Always run `what-if` before deploying:

```bash
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters environments/prod.bicepparam
```

---

## Security Checklist

- [ ] `@secure()` on all secret parameters
- [ ] No default values on secure parameters
- [ ] Key Vault references for secrets in App Settings
- [ ] Managed Identity (system-assigned) on all compute
- [ ] Private Endpoints for all PaaS resources
- [ ] TLS 1.2 minimum on all services
- [ ] RBAC authorization on Key Vault (not access policies)
- [ ] Diagnostic settings on all resources → Log Analytics
- [ ] NSGs on all subnets with deny-all-inbound default
- [ ] OIDC for CI/CD (no client secrets)

## Review Checklist

| Area | Check |
|------|-------|
| **Structure** | Modules for reusable components? Env params separated? |
| **Types** | Proper decorators (`@allowed`, `@minLength`, `@secure`)? |
| **Naming** | Consistent naming convention? Azure abbreviations? |
| **Outputs** | All downstream-needed values exported? Described? |
| **Linting** | `bicepconfig.json` configured? All warnings resolved? |
| **AVM** | Using Azure Verified Modules where available? |
| **Idempotency** | Safe to re-run? No `utcNow()` in resource properties? |
| **Dependencies** | Implicit only? No unnecessary `dependsOn`? |

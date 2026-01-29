# Azure Network Security Perimeter (NSP) for Azure Databricks Serverless

## Overview

This ARM template deploys an Azure Network Security Perimeter (NSP) configured for Azure Databricks serverless compute. NSP creates a logical isolation boundary for your PaaS resources, enabling centralized network access management using a simplified rule set.

---

## Table of Contents

- [Introduction](#introduction)
- [Architecture](#architecture)
- [Key Benefits](#key-benefits)
- [Supported Azure Services](#supported-azure-services)
- [Prerequisites](#prerequisites)
- [Template Structure](#template-structure)
- [Parameters Reference](#parameters-reference)
- [Deployment Guide](#deployment-guide)
- [Parameter File Examples](#parameter-file-examples)
- [Verification Steps](#verification-steps)
- [Access Modes](#access-modes)
- [Regional Restrictions](#regional-restrictions)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## Introduction

Azure Network Security Perimeter (NSP) is an Azure-native feature that allows you to:

- Create logical isolation boundaries for PaaS resources
- Centrally manage network access using simplified rule sets
- Replace complex IP address and subnet ID management
- Control access from Azure Databricks serverless compute to your Azure resources

### What This Template Does

1. **Creates a Network Security Perimeter** - The main NSP resource
2. **Creates an NSP Profile** - A configuration profile within the NSP
3. **Creates an Inbound Access Rule** - Allows traffic from `AzureDatabricksServerless` service tag
4. **Associates Azure Resources** - Optionally associates storage accounts, Key Vaults, SQL servers, and other resources with the NSP

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Azure Subscription                                   │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                 Network Security Perimeter (NSP)                      │  │
│  │                                                                       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
│  │  │                      NSP Profile                                 │ │  │
│  │  │                                                                  │ │  │
│  │  │  ┌────────────────────────────────────────────────────────────┐│ │  │
│  │  │  │              Inbound Access Rule                           ││ │  │
│  │  │  │                                                            ││ │  │
│  │  │  │  Source: AzureDatabricksServerless (Service Tag)          ││ │  │
│  │  │  │  Direction: Inbound                                        ││ │  │
│  │  │  └────────────────────────────────────────────────────────────┘│ │  │
│  │  └─────────────────────────────────────────────────────────────────┘ │  │
│  │                                                                       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │  │
│  │  │                  Resource Associations                          │ │  │
│  │  │                                                                  │ │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │ │  │
│  │  │  │   Storage    │  │  Key Vault   │  │  SQL Server  │          │ │  │
│  │  │  │   Account    │  │              │  │              │          │ │  │
│  │  │  │  (Learning)  │  │  (Learning)  │  │  (Learning)  │          │ │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │ │  │
│  │  │                                                                  │ │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │ │  │
│  │  │  │  Event Hub   │  │  Cosmos DB   │  │ Service Bus  │          │ │  │
│  │  │  │              │  │              │  │              │          │ │  │
│  │  │  │  (Learning)  │  │  (Learning)  │  │  (Learning)  │          │ │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │ │  │
│  │  └─────────────────────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                                    ▲                                        │
│                                    │                                        │
│                          Inbound Traffic                                    │
│                                    │                                        │
│  ┌─────────────────────────────────┴─────────────────────────────────────┐  │
│  │              Azure Databricks Serverless Compute                      │  │
│  │                                                                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │  │
│  │  │ SQL         │  │ Jobs        │  │ Notebooks   │  │ Model       │  │  │
│  │  │ Warehouses  │  │             │  │             │  │ Serving     │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Benefits

| Benefit | Description |
|---------|-------------|
| **Cost Savings** | Traffic sent over service endpoints stays on the Azure backbone and incurs no data processing charges |
| **Simplified Management** | Use the `AzureDatabricksServerless` service tag to manage access globally without managing IP addresses |
| **Centralized Access Control** | Manage security policies across multiple resource types within a single NSP profile |
| **Regional Flexibility** | Restrict access to specific Azure regions using region-specific service tags (e.g., `AzureDatabricksServerless.EastUS2`) |
| **No IP Management** | Service tags automatically update when Azure Databricks adds new IP ranges |

---

## Supported Azure Services

NSP supports secure connections from Databricks serverless compute to:

### Data & Analytics
- Azure Storage (including ADLS Gen2)
- Azure SQL Database
- Synapse Analytics
- Cosmos DB
- MariaDB

### Security & Apps
- Key Vault
- App Service
- Cognitive Services

### Messaging & DevOps
- Event Hubs
- Service Bus
- Container Registry

---

## Prerequisites

Before deploying this template, ensure you have:

| Requirement | Description |
|-------------|-------------|
| **Azure Databricks Account Admin** | You must be an Azure Databricks account administrator |
| **Contributor/Owner Permissions** | Required on the Azure resources you want to configure |
| **NSP Creation Permissions** | Permission to create network security perimeter resources in your subscription |
| **Regional Alignment** | Your Databricks workspace and Azure resources should be in the same Azure region |

---

## Template Structure

```
├── ARM Template (nsp-template.json)
│   ├── Parameters
│   │   ├── NSP Configuration
│   │   ├── Profile Configuration
│   │   ├── Access Rule Configuration
│   │   └── Resource Association Flags
│   ├── Variables
│   │   ├── API Version
│   │   ├── Service Tag Logic
│   │   └── Resource IDs
│   ├── Resources
│   │   ├── Network Security Perimeter
│   │   ├── NSP Profile
│   │   ├── Inbound Access Rule
│   │   └── Resource Associations (conditional)
│   └── Outputs
│       ├── NSP Resource ID
│       ├── NSP Profile Resource ID
│       └── Service Tag Used
```

---

## Parameters Reference

### Core NSP Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `nspName` | String | Yes | - | Name of the Network Security Perimeter |
| `location` | String | No | Resource Group Location | Azure region for the NSP |
| `nspProfileName` | String | No | `databricks-profile` | Name of the NSP Profile |
| `inboundRuleName` | String | No | `allow-databricks-serverless` | Name of the inbound access rule |

### Access Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `nspAccessMode` | String | No | `Learning` | Access mode: `Learning` (Transition) or `Enforced` |
| `restrictToSpecificRegion` | Bool | No | `false` | Restrict to specific Azure region |
| `serverlessRegion` | String | No | `""` | Region name if restricting (e.g., `EastUS2`) |

### Resource Association Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `associateStorageAccount` | Bool | `false` | Associate a storage account |
| `storageAccountResourceId` | String | `""` | Full resource ID of storage account |
| `associateKeyVault` | Bool | `false` | Associate a Key Vault |
| `keyVaultResourceId` | String | `""` | Full resource ID of Key Vault |
| `associateSqlServer` | Bool | `false` | Associate an Azure SQL Server |
| `sqlServerResourceId` | String | `""` | Full resource ID of SQL Server |
| `associateEventHub` | Bool | `false` | Associate an Event Hub namespace |
| `eventHubResourceId` | String | `""` | Full resource ID of Event Hub |
| `associateCosmosDb` | Bool | `false` | Associate a Cosmos DB account |
| `cosmosDbResourceId` | String | `""` | Full resource ID of Cosmos DB |
| `associateServiceBus` | Bool | `false` | Associate a Service Bus namespace |
| `serviceBusResourceId` | String | `""` | Full resource ID of Service Bus |
| `associateContainerRegistry` | Bool | `false` | Associate a Container Registry |
| `containerRegistryResourceId` | String | `""` | Full resource ID of Container Registry |
| `associateCognitiveServices` | Bool | `false` | Associate Cognitive Services |
| `cognitiveServicesResourceId` | String | `""` | Full resource ID of Cognitive Services |

---

## Deployment Guide

### Method 1: Azure Portal

1. Navigate to **Deploy a custom template** in the Azure Portal
2. Click **Build your own template in the editor**
3. Paste the ARM template JSON
4. Click **Save**
5. Fill in the required parameters
6. Click **Review + create**
7. Click **Create**

### Method 2: Azure CLI

```bash
# Set variables
RESOURCE_GROUP="your-resource-group"
DEPLOYMENT_NAME="nsp-deployment"
TEMPLATE_FILE="nsp-template.json"
PARAMETERS_FILE="nsp-parameters.json"

# Deploy the template
az deployment group create \
    --resource-group $RESOURCE_GROUP \
    --name $DEPLOYMENT_NAME \
    --template-file $TEMPLATE_FILE \
    --parameters @$PARAMETERS_FILE
```

### Method 3: Azure PowerShell

```powershell
# Set variables
$ResourceGroup = "your-resource-group"
$DeploymentName = "nsp-deployment"
$TemplateFile = "nsp-template.json"
$ParametersFile = "nsp-parameters.json"

# Deploy the template
New-AzResourceGroupDeployment `
    -ResourceGroupName $ResourceGroup `
    -Name $DeploymentName `
    -TemplateFile $TemplateFile `
    -TemplateParameterFile $ParametersFile
```

### Method 4: Azure DevOps Pipeline

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: AzureResourceManagerTemplateDeployment@3
    inputs:
      deploymentScope: 'Resource Group'
      azureResourceManagerConnection: 'your-service-connection'
      subscriptionId: 'your-subscription-id'
      action: 'Create Or Update Resource Group'
      resourceGroupName: 'your-resource-group'
      location: 'East US 2'
      templateLocation: 'Linked artifact'
      csmFile: '$(Build.SourcesDirectory)/nsp-template.json'
      csmParametersFile: '$(Build.SourcesDirectory)/nsp-parameters.json'
      deploymentMode: 'Incremental'
```

---

## Parameter File Examples

### Example 1: Basic NSP with Storage Account

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nspName": {
            "value": "databricks-nsp"
        },
        "location": {
            "value": "eastus2"
        },
        "nspProfileName": {
            "value": "databricks-profile"
        },
        "nspAccessMode": {
            "value": "Learning"
        },
        "associateStorageAccount": {
            "value": true
        },
        "storageAccountResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.Storage/storageAccounts/mystorageaccount"
        }
    }
}
```

### Example 2: NSP with Region-Specific Restriction

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nspName": {
            "value": "databricks-nsp-eastus2"
        },
        "location": {
            "value": "eastus2"
        },
        "nspProfileName": {
            "value": "databricks-profile-regional"
        },
        "restrictToSpecificRegion": {
            "value": true
        },
        "serverlessRegion": {
            "value": "EastUS2"
        },
        "nspAccessMode": {
            "value": "Learning"
        },
        "associateStorageAccount": {
            "value": true
        },
        "storageAccountResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.Storage/storageAccounts/mystorageaccount"
        }
    }
}
```

### Example 3: NSP with Multiple Resources

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nspName": {
            "value": "databricks-nsp-full"
        },
        "location": {
            "value": "eastus2"
        },
        "nspProfileName": {
            "value": "databricks-profile-full"
        },
        "nspAccessMode": {
            "value": "Learning"
        },
        "associateStorageAccount": {
            "value": true
        },
        "storageAccountResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.Storage/storageAccounts/mystorageaccount"
        },
        "associateKeyVault": {
            "value": true
        },
        "keyVaultResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.KeyVault/vaults/mykeyvault"
        },
        "associateSqlServer": {
            "value": true
        },
        "sqlServerResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.Sql/servers/mysqlserver"
        },
        "associateEventHub": {
            "value": true
        },
        "eventHubResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-rg/providers/Microsoft.EventHub/namespaces/myeventhub"
        }
    }
}
```

### Example 4: Production Configuration (Enforced Mode)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nspName": {
            "value": "databricks-nsp-prod"
        },
        "location": {
            "value": "eastus2"
        },
        "nspProfileName": {
            "value": "databricks-profile-prod"
        },
        "nspAccessMode": {
            "value": "Enforced"
        },
        "restrictToSpecificRegion": {
            "value": true
        },
        "serverlessRegion": {
            "value": "EastUS2"
        },
        "associateStorageAccount": {
            "value": true
        },
        "storageAccountResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/prod-rg/providers/Microsoft.Storage/storageAccounts/prodstorageaccount"
        },
        "associateKeyVault": {
            "value": true
        },
        "keyVaultResourceId": {
            "value": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/prod-rg/providers/Microsoft.KeyVault/vaults/prodkeyvault"
        }
    }
}
```

---

## Verification Steps

After deployment, verify your NSP configuration:

### Step 1: Verify NSP Creation

```bash
# Azure CLI
az network perimeter show \
    --name "databricks-nsp" \
    --resource-group "your-resource-group"
```

### Step 2: Verify Resource Association

1. Navigate to your Azure resource in the Azure Portal
2. Go to **Security + networking > Networking**
3. Verify that the resource shows an association with your network security perimeter
4. Verify that the status shows **Learning** (Transition) or **Enforced** mode

### Step 3: Verify Inbound Rules

1. Navigate to your NSP in the Azure Portal
2. Go to **Settings > Profiles**
3. Select your profile
4. Click **Inbound access rules**
5.

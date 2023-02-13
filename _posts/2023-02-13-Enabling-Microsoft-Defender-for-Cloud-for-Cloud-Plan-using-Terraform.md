---
layout: post

title: Enabling Microsoft Defender for Cloud Plan using Terraform
subtitle: Securing Azure Cloud Environment with Terraform and Microsoft Defender
cover-img: /assets/img/tf-storage.jpg
thumbnail-img: /assets/img/tf-storage.jpg
share-img: /assets/img/tf-storage.jpg
tags: [Azure security, Microsoft Defender for Cloud, Terraform, Infrastructure as Code, Azure resources, AzureRM Provider, Security API, MDC , IAC]
---


# Enabling Microsoft Defender for Cloud Plan using Terraform

Terraform is a widely adopted **Infrastructure as Code (IAC)** tool that is utilized extensively by enterprises that prioritize Azure security. It is recommended to deploy all Azure security-related configurations using Terraform to ensure consistency.  

## Review Enabled Defender Plans from Azure Portal

We can review the existing Environmental settings by 
1. Sign in to the Azure portal.

2. Search for and select Microsoft Defender for Cloud.

3. In the Defender for Cloud menu, select Environment settings.

4. Select the subscription or workspace that you want to protect.


![Defender for Cloud Before ](/assets/img/tf-storage-mdc-setting-before.jpg)

You can also review the same by calling [Defender for Cloud API](https://learn.microsoft.com/en-us/rest/api/defenderforcloud/pricings/list)
`GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Security/pricings?api-version=2022-03-01`

You will see  response similar to this. I have truncated the response. 
```json
  {
  "value": [
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/VirtualMachines",
      "name": "VirtualMachines",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "subPlan": "P2",
        "pricingTier": "Standard",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/SqlServers",
      "name": "SqlServers",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/AppServices",
      "name": "AppServices",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/StorageAccounts",
      "name": "StorageAccounts",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/SqlServerVirtualMachines",
      "name": "SqlServerVirtualMachines",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/KubernetesService",
      "name": "KubernetesService",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S",
        "deprecated": true,
        "replacedBy": [
          "Containers"
        ]
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/ContainerRegistry",
      "name": "ContainerRegistry",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S",
        "deprecated": true,
        "replacedBy": [
          "Containers"
        ]
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/KeyVaults",
      "name": "KeyVaults",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    },
    {
      "id": "/subscriptions/3bfeb287-1a11-406b-8e99-***************/providers/Microsoft.Security/pricings/Dns",
      "name": "Dns",
      "type": "Microsoft.Security/pricings",
      "properties": {
        "pricingTier": "Free",
        "freeTrialRemainingTime": "PT0S"
      }
    }
    }
  ]
}
```

The following files are present in the Terraform workspace:
- `main.tf`: This file contains the desired state configuration of your MDC deployment, including the deployment of MDC.
- `variables.tf`: This file holds values that are specific to different environments, such as development or production.
- `outputs.tf`: This file declares information that becomes available only after deployment.
- `data.tf`: This file retrieves information from Azure APIs, which is used in the configuration.

## Terraform Resources for MDC

Details about the security center-related resources can be found in the [AzureRM Provider documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/advanced_threat_protection), which is aggregated under the "Security Center" section. The resource required to enable the Microsoft Defender plan in the subscription is used in the following code snippets.
  
#### data.tf
```terraform
data "azurerm_subscription" "current" {}
```
#### main.tf
```terraform
terraform {
  required_version = ">=0.12"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.0"
    }
  }
}
provider "azurerm" {
  features {}
}

resource "azurerm_security_center_subscription_pricing" "mdc_servers" {
  tier          = "Standard"
  resource_type = "StorageAccounts"
}
resource "azurerm_security_center_subscription_pricing" "mdc_SqlServers" {
  tier          = "Standard"
  resource_type = "SqlServers"
}
resource "azurerm_security_center_subscription_pricing" "mdc_AppServices" {
  tier          = "Standard"
  resource_type = "AppServices"
}


resource "azurerm_security_center_subscription_pricing" "mdc_StorageAccounts" {
  tier          = "Standard"
  resource_type = "StorageAccounts"
}

resource "azurerm_security_center_subscription_pricing" "mdc_SqlServerVirtualMachines" {
  tier          = "Standard"
  resource_type = "SqlServerVirtualMachines"
}

resource "azurerm_security_center_subscription_pricing" "mdc_KeyVaults" {
  tier          = "Standard"
  resource_type = "KeyVaults"
}


resource "azurerm_security_center_subscription_pricing" "mdc_Dns" {
  tier          = "Standard"
  resource_type = "Dns"
}

resource "azurerm_security_center_subscription_pricing" "mdc_Arm" {
  tier          = "Standard"
  resource_type = "Arm"
}



resource "azurerm_security_center_subscription_pricing" "mdc_OpenSourceRelationalDatabases" {
  tier          = "Standard"
  resource_type = "OpenSourceRelationalDatabases"
}



resource "azurerm_security_center_subscription_pricing" "mdc_Containers" {
  tier          = "Standard"
  resource_type = "Containers"
}



```
In the examples above, we are enabling Defender plans in our Subscription.  For other plans, check out the [Terraform documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/security_center_subscription_pricing).
Run the following three commands to deploy the configuration from main.tf in order.  
>Terraform init  
  
This command downloads the Terraform files to a subfolder called `.terraform`, where it stores the Azure RM provider.


>Terraform plan  

This command shows what changes will be made, but no changes will be made to Azure.

>Terraform apply  

This command applies the changes from main.tf to your Azure environment.  
You can verify the configuration deployed in Azure Portal.  
![Mdfc-setting](/assets/img/tf-storage-mdc-setting-after.jpg)



In conclusion, Terraform is a popular Infrastructure as Code tool for deploying Azure resources, especially for enterprises focused on Azure security. This guide provides information on how to deploy Microsoft Defender for Cloud (MDC) plans  using Terraform and the necessary files in the workspace. Terraform is recommended for managing MDC exclusively to ensure consistency, and its use is supported by the AzureRM Provider documentation.


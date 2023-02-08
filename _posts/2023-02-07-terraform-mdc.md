---
layout: post

title: Deploying Microsoft Defender for Cloud for Storage using Terraform
subtitle: A Comprehensive Guide for Enterprise Azure Security
cover-img: /assets/img/tf-storage.jpg
thumbnail-img: /assets/img/tf-storage.jpg
share-img: /assets/img/tf-storage.jpg
tags: [Azure security, Microsoft Defender for Cloud, Terraform, Infrastructure as Code, Azure resources, AzureRM Provider]
---


# Deploying Microsoft Defender for Cloud (MDC) for Storage using Terraform

Terraform is a widely adopted **Infrastructure as Code (IAC)** tool that is utilized extensively by enterprises that prioritize Azure security. It is recommended to deploy all Azure security-related configurations using Terraform to ensure consistency.

The following files are present in the Terraform workspace:
- `main.tf`: This file contains the desired state configuration of your MDC deployment, including the deployment of MDC.
- `variables.tf`: This file holds values that are specific to different environments, such as development or production.
- `outputs.tf`: This file declares information that becomes available only after deployment.
- `data.tf`: This file retrieves information from Azure APIs, which is used in the configuration.

## Terraform Resources for MDC

Details about the security center-related resources can be found in the [AzureRM Provider documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/advanced_threat_protection), which is aggregated under the "Security Center" section. The resource required to enable the Microsoft Defender for Storage plan in the subscription is used in the following code snippets.
  
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
```
In the examples above, we are enabling Defender for Storage in our Subscription For other plans, check out the Terraform documentation.
Run the following three commands to deploy the configuration from main.tf in order.  
>Terraform init  
  
This command downloads the Terraform files to a subfolder called `.terraform`, where it stores the Azure RM provider.


>Terraform plan  

This command shows what changes will be made, but no changes will be made to Azure.

>Terraform apply  

This command applies the changes from main.tf to your Azure environment.  
You can verify the configuration deployed in Azure Portal.  
![Mdfc-setting](/assets/img/mdc-storage.jpg)



In conclusion, Terraform is a popular Infrastructure as Code tool for deploying Azure resources, especially for enterprises focused on Azure security. This guide provides information on how to deploy Microsoft Defender for Cloud (MDC) for storage using Terraform and the necessary files in the workspace. Terraform is recommended for managing MDC exclusively to ensure consistency, and its use is supported by the AzureRM Provider documentation.


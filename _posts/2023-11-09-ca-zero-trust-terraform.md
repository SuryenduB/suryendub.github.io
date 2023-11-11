---
layout: post
title: CI / CD Deployment of  Conditional Access Policies for a Zero Trust Architecture Framework using Terraform and GitHub Actions
subtitle :   Deploy , Manage and Monitor Conditional Access using Terraform and GitHub Actions
cover-img: /assets/img/CABlog16.png
thumbnail-img: /assets/img/CABlog16.png
share-img: /assets/img/CABlog16.png
tags: [ Azure Active Directory, EntraID, MicrosoftEntra, Security, ZeroTrust, API, OAuth, ConditionalAccess]

---

# Deploy Conditional Access Policies for a Zero Trust Architecture Framework using Terraform and GitHub Actions

## Table of Contents

- [Introduction](#introduction)  
- [The Framework](#the-framework)  
- [Conditional Access Architecture](#conditional-access-architecture)  
- [Import Existing Conditional Access Policies to Terraform](#import-existing-conditional-access-policies-to-terraform)
  - [Config Driven Import](#config-driven-import)
    - [List Conditional Access Policies](#list-conditional-access-policies)
    - [List of Policies for Import](#list-conditional-access-policies)
    - [Add Exclusion to Conditional Access Policies](#add-exclusion-to-conditional-access-policies)
- [The Source Repo](#the-source-repo)
- [Azure AD Group Resource](#azure-ad-group-resource)
- [Conditional Access Policy Resource](#conditional-access-policy-resource)
- [Named Location Resource](#named-location-resource)
- [Ring Base Deployment](#ring-base-deployment)
  - [Steps to deploy a new CA Policy](#steps-to-deploy-a-new-ca-policy)
- [Use GitHub Actions for Automated Ring Base Deployment of new CA Policy](#use-github-actions-for-automated-ring-base-deployment-of-new-ca-policy)
- [Final Thoughts](#final-thoughts)
- [References](#references)

## Introduction

I have not managed to write something new and my mind has progressively started to become more and more arid. Lately, I had to think about the [Conditional Access Policy](https:/learn.microsoft.com/en-us/entra/identity/conditional-access/overview) and what is the best way to deploy and manage them. I have been working on a project where we are trying to automate the deployment of Conditional Access Policies and the monitoring of the policies. Inimitable [Claus Jespersen](https:/www.linkedin.com/in/claus-jespersen-25b0422/) has created an extraordinary framework [Conditional Access for Zero Trust Resources](https:/github.com/microsoft/ConditionalAccessforZeroTrustResources) that ties together all the loose ends for **Conditional Access for Zero Trust**. We want to have continuous integration and continuous deployment enabled, meaning that 
each time a change to the CA policies has been committed/approved, we want to 
automatically deploy the new CA policies. Also even if there are no changes, we want to 
ensure that the running set of policies have been changed manually using GUI 
and if so, align with the approved policies in the repository.

## The Framework

Anyone working in this domain can benefit a lot from the [Microsoft Azure AD Conditional Access principles and guidance](https:/github.com/microsoft/ConditionalAccessforZeroTrustResources/blob/main/ConditionalAccessGovernanceAndPrinciplesforZeroTrust%20October%202023.pdf).

For the benefit of the reader let me try to summarize a few of the core concepts of this framework. 
> "A better approach is to structure policies related to common access needs and contain a set of 
access needs in a persona, representing these needs for various users who have the same needs."

Different Identities present in the organization can be broadly classified into categories. As your organization grows the complexity of the identities also grows and you end up with different categories of personas. However, all the organizations can be broadly classified into the following categories.

![Persona Groups](/assets/img/Persona-Groups.jpg)

For each of the Identity Personas, we can create policies targeted to the specific persona. The policies can be further classified into the following categories.

| Policy Type | Description |
| ----------- | ----------- |
| Base Protection | The base protection is the default policy for all apps for users of the given persona. |
| Identity Protection | This policy deals with Azure AD Identity Protection for the Personna |
| Data Protection| Policies that protect data as an extra layer on top of the base protection |
| Attack Surface Reduction  | Policy to mitigate against various attacks. |
| Compliance | Policies to ensure compliance with various regulations. |

Not only Personas and Policy Type this framework also guides various other aspects like best practices, exclusions, and deployments (more of this later).

## Conditional Access Architecture

There are two main Conditional Access architectures: Targeted and Zero Trust.

**Targeted architecture** only targets individual apps in CA policies that you want to protect. This means that endpoints like the device-login endpoint are not subjective to the CA policies and hence will continue to work. However, the challenge of using this architecture is that you may forget to protect all cloud apps.

**Zero Trust CA architecture** is the one that best fits the principles of Zero Trust. Choosing "All cloud apps" in a CA policy implies that all endpoints are protected by the given grant controls, like known users and known or compliant devices. However, the policy applies not only to the endpoints/apps that support Conditional Access but any endpoint that the end-user interacts with.

Zero Trust CA architecture is the recommended architecture. It is better aligned with a Zero Trust strategy and will automatically protect any new app.

## Import Existing Conditional Access Policies to Terraform

Managing Conditional Access policies in Entra ID at scale can be a real hassle. The GUI-based management tools were not designed to perform any kind of configuration in bulk. For Unified Management of Conditional Access Management, it is important to use a tool that allows us to not only create new Conditional Access Policies but at the same time manage the existing Conditional Access Policies.

## Config Driven Import

In my Test Tenant, we have a few Conditional Access Policies that we will be importing into Terraform. The following is the list of Conditional Access Policies that we will be importing into Terraform. On a Side note,  recently  Microsoft has announced that they are going to create the following [default Conditional Access Policies](https:/www.microsoft.com/en-us/security/blog/2023/11/06/automatic-conditional-access-policies-in-microsoft-entra-streamline-identity-protection/#:~:text=Microsoft%2Dmanaged%20Conditional%20Access%20policies%20provide%20clear%2C%20self%2Ddeploying,but%20we're%20starting%20simple.) in all tenant.  It is important for the organizations to calibrate the above policies and you should exclude the breakglass accounts from the above policies and enable it. I have selected the **Microsoft-Managed Policies** in my test tenant to showcase, how we can import existing CA Policy using Terraform Import. Later we will use the imported policies to modify as per our needs and add new policies based on the **Conditional Access for Zero Trust** framework.

First Start with Blank Configuration in VSCode and just add the provider block for Azure AD Provider. The ultimate goal of our execution is to be able to deploy and Manage the Conditional Access Policy from the CI/CD pipeline. We will need to configure the [remote backend](https:/learn.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform) for state file management.

```hcl
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "=2.45.0"
    }
  }
    backend "azurerm" {
    storage_account_name = "iamstgca"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }

}
# Configure the Microsoft Azure Provider
provider "azuread" {

}

```

Now we will define the import block for the Conditional Access Policy. The import block will have the following attributes

1. Resource Type we need to import
2. Object (Resource ID) of the resource. conditional Access Policy / Authentication Strength Policy we need to import.

You can get the resource ID of your conditional Access Policies in several ways. I normally use Powershell for Microsoft Graph Operations.

```powershell
Invoke-MgGraphRequest -Uri "https:/graph.microsoft.com/v1.0/policies/conditionalAccessPolicies?$select=id,displayName" | Select-Object -ExpandProperty Value | Select Id , displayName
```

### **List Conditional Access Policies**

![Conditional Access Policies](/assets/img/CABlog1.png)

Now that we have the ID of our Conditional Access Policy we can use the following command to import the Conditional Access Policy into Terraform. As you can see we can also import other resources as well as we are doing in the following example. We have imported the Authentication Strength Policy as well as the Conditional Access Policy.

```hcl
import {
   to =  azuread_authentication_strength_policy.fido2_security_key
   id =  "ad86698e-4ac4-4643-8d27-99f980960382"
}

import {
   to =  azuread_conditional_access_policy. Require_multifactor_authentication_Microsoft_Admin_Portals
   id =  "7298af73-08b1-482f-af63-d24879a037a7"
}
```

Now that we are set up and ready to import, we can run the following command to generate the TF file for us to inspect.

> ```terraform plan -generate-config-out azuread_conditional_access_policy.tf``` 

In this case, we see that we need to fix the following error.

![Error](/assets/img/CABlog2.png)

We can fix this by first removing the ```included_user_actions``` block from the policy.

![Error](/assets/img/CABlog3.png)



![Error](/assets/img/CABlog4.png)

Once we are happy with the TF file we can simply run apply and these resources will be imported into our state file.

> ```terraform apply -auto-approve```

We can now check this file and validate that the resources are imported into the state file.



>[Note] We have imported existing resources created outside the scope of Terraform and we can now manage these resources using Terraform.

### Add Exclusion to Conditional Access Policies

As per the Zero Trust Access Framework for Conditional Access we are trying to deploy we must always exclude resources that are used for breakglass scenarios. In our case, we have a breakglass account that we want to exclude from all Conditional Access Policies. We can do this by adding the following block to our Conditional Access Policy. Here ```d3f027e6-6e7a-4643-83e9-d5b423a925d5``` is the user ID of the breakglass account.

```hcl

  users {
      excluded_groups = []
      excluded_roles  = []
      excluded_users  = ["c60de183-da08-432d-a40c-b1eae564fb98","d3f027e6-6e7a-4643-83e9-d5b423a925d5"]
      included_groups = []
      included_roles  = []
      included_users  = ["All"]
    }
```

We need to run the `terraform plan ` and `terraform apply` respectively to generate the configuration.

![Terraform Plan Output](/assets/img/CABlog7.png).

You can verify Conditional Access Policy is updated in the EntraID portal.

## The Source Repo

Now that we have managed to import the Existing Conditional Access Policies in our state file we will deploy new Conditional Access Policies based on our **Zero Trust Framework**. We will be using [this repository](https:/github.com/SuryenduB/ConditionalAccessforZeroTrustResourcesTerraform) as the source repo for our Terraform Code and CI/CD Pipeline. The repository has the following structure.

### **Azure AD Group Resource**

In the  [Introduction](#introduction) section, I have extensively written about how the Zero Trust Framework and Conditional Access Deployment are based on persona groups. However, we must remember this is an ever-changing need and it will continue to spawn as organizations expand. We need to separate this from the main code.

> **azuread_group.tf**

A small recap of the persona groups for the Zero Trust Framework.

- Ring-Based Groups for Deployment
- Persona-Based Groups for Exclusions
- Persona-Based Dynamic Group for Policy Assignment

One of the core goals here is to separate the configuration from business logic. In this case, all the different persona groups are configured in the local block, and the logic to create Persona Groups follows the **D-R-Y** principal. 

```hcl

```hcl
locals {
  persona_types = [
    "Internals",
    "Developers",
    "Externals",
    "Guests",
    "GuestAdmins",
    "CorpServiceAccounts",
    "Admins",
    "WorkloadIdentities",
  ]
  policy_types = [
    "BaseProtection",
    "AppProtection",
    "IdentityProtection",
    "DataProtection",
    "AttackSurfaceReduction",
  ]
  ring_types = [
    "Ring0",
    "Ring1",
    "Ring2",
    "Ring3",
  ]
  persona_type_policy_type = distinct(flatten([
    for persona_type in local.persona_types : [
      for policy_type in local.policy_types : {
        persona_type = persona_type
        policy_type  = policy_type
      }
    ]
  ]))

  persona_type_ring_type = distinct(flatten([
    for persona_type in local.persona_types : [
      for ring_type in local.ring_types : {
        persona_type = persona_type
        ring_type    = ring_type
      }
    ]
  ]))
```

We will be using the following code to generate the ring groups for each Persona Type.

```hcl
resource "azuread_group" "CA-Persona-Rings" {
  # We need a map to use for_each, so we convert our list into a map by adding a unique key:
  for_each         = { for entry in local.persona_type_ring_type : "${entry.persona_type}.${entry.ring_type}" => entry }
  display_name     = "CA-Persona-${each.value.persona_type}-${each.value.ring_type}"
  security_enabled = true
  description      = "Manually managed by Conditional Access administrators"
}
```

This will generate the Ring-based group for each Persona Type.

![Ring Based Groups](/assets/img/CABlog9.png)


We will be using the following code to generate the persona groups. Now each of these persona groups are dynamic groups. For Example, I have considered for the  Internal Employees to have 5 digits Employee ID and External Users to have `ext` in their UPN. You can modify it to your specific organizational needs.

```hcl
resource "azuread_group" "CA-Persona-Groups" {
  # We need a map to use for_each, so we convert our list into a map by adding a unique key:
  for_each         = toset(local.persona_types)
  display_name     = "CA-Persona-${each.value}"
  security_enabled = true
  description      = "Manually managed by Conditional Access administrators"
  # if each.value == "Internals" {}
  types = ["DynamicMembership"]
  dynamic_membership {
    enabled = true
    rule    = each.value == "Internals" ? "user.employeeid -match \"\\d{5}$\"" : each.value == "Externals" ? "user.userPrincipalName -contains \"ext\"" : each.value == "Guests" ? "user.userType  -contains \"Guest\"" : each.value == "CorpServiceAccounts" ? "user.userPrincipalName -contains \"serviceAccounts\"" : each.value == "Admins" ? "user.userPrincipalName -contains \"admin\"" : each.value == "GuestAdmins" ? "user.userType  -contains \"Guest\" -and user.userPrincipalName -contains \"admins\" " : "user.department -match \"${each.value}\""

  }
}
```

![Persona Based Groups: Internal](/assets/img/CABlog10.png)

Similar to Ring-based groups we will be using the following code to generate the persona groups for exclusions. For each policy type, we will add specific exclusion groups

```hcl
resource "azuread_group" "CA-Persona-Groups-Exclusions" {
  # We need a map to use for_each, so we convert our list into a map by adding a unique key:
  for_each         = { for entry in local.persona_type_policy_type : "${entry.persona_type}.${entry.policy_type}" => entry }
  display_name     = "CA-Persona-${each.value.persona_type}-${each.value.policy_type}-exclusions"
  security_enabled = true
  description      = "Manually managed by Conditional Access administrators"

}
```

![Persona Based Groups : Exclusions](/assets/img/CABlog11.png)

### Conditional Access Policy Resource

> **azuread_conditional_access_policy.tf** 

We will update the previously generated **azuread_conditional_access_policy.tf** file with the Zero Trust-based conditional Access Policies from our Zero Trust Framework. Zero Trust framework suggests to structure conditional Access Policies according to the following areas :

- Global protection (CA001-CA099)
- Admins protection (CA100-CA199)
- Internals user protection (CA200-CA299)
- Externals user protection (CA300-CA399)
- Guests user protection (CA400-CA499)
- GuestAdmins user admins protection (CA500-CA599)
- Microsoft365ServiceAccounts (CA600-CA699)
- AzureServiceAccounts (CA700-CA799)
- CorpServiceAccounts (CA800-CA899)
- WorkloadIdentities (CA900-CA999)
- Developer (CA1000-CA1099)
I have included few of the sample policies in this blog and you will get the remaining policies in the source **[repo](#the-source-repo)**.

**Internals Base Protection**: Require known user and Compliant or Azure AD Hybrid Joined device from any device.

```hcl
resource "azuread_conditional_access_policy" "CA200-Internals-BaseProtection-AllApps-AnyPlatform-CompliantorAADHJ" {
  display_name = "CA200-Internals-BaseProtection-AllApps-AnyPlatform-CompliantorAADHJ"
  state        = "enabledForReportingButNotEnforced"
 conditions {
    applications {
      included_applications = ["All"]
      excluded_applications = [data.azuread_service_principal.intune.client_id]
    }
    platforms {
      included_platforms = ["all"]
    }
users {
      included_groups = [azuread_group.CA-Persona-Groups["Internals"].id]
      excluded_groups = [azuread_group.breakglass.id, azuread_group.CA-Persona-Groups-Exclusions["Internals.BaseProtection"].id]
    }
    client_app_types = ["all"]
  }
  grant_controls {
    operator          = "OR"
    built_in_controls = ["compliantDevice", "domainJoinedDevice"]
  }
}
```

**Internals Identity Protection**: For Users with **High Risk Level** in Azure AD Identity Protection, users will need to authenticate with MFA and Password Change.

```hcl
resource "azuread_conditional_access_policy" "CA202-Internals-IdentityProtection-AllApps-AnyPlatform-MFAandPWDforHighUserRisk" {
  display_name = "CA202-Internals-IdentityProtection-AllApps-AnyPlatform-MFAandPWDforHighUserRisk"
  state        = "enabledForReportingButNotEnforced"
  conditions {
    applications {
      included_applications = ["All"]
    }
    client_app_types = ["all"]
    user_risk_levels = ["high"]
    users {
      included_groups = [azuread_group.CA-Persona-Groups["Internals"].id]
      excluded_groups = [azuread_group.breakglass.id, azuread_group.CA-Persona-Groups-Exclusions["Internals.IdentityProtection"].id]
    }
  }
  grant_controls {
    operator          = "AND"
    built_in_controls = ["passwordChange", "mfa"]
  }
}
```

Notice how we have used the group resources generated to apply and exclude the policies in different personas. You can find the complete code in the repo mentioned above.

### **Named Location Resource** 

> **azuread_named_location.tf**: In a few of the Conditional Access Policies we have used Named Location. Named Location is a resource that can be used to define a set of IP addresses or countries that can be used in Conditional Access Policies. We will be using the following code to generate the Named Location Resource.

- CA800-CorpServiceAccounts-BaseProtection-AllApps-AnyPlatform-BlockUntrustedLocations
- CA900-WorkloadIdentities-BaseProtection-AllApps-AnyPlatform-BlockUntrustedLocations

```hcl


```hcl
resource "azuread_named_location" "AzureVnet-ip" {
  display_name = "AzureVnet-ip IP Named Location"
  ip {
    ip_ranges = [
      "1.1.1.1/32",
      "2.2.2.2/32"
    ]
    trusted = true
  }
}
resource "azuread_named_location" "TrustedLocation" {
  display_name = "Trusted Country"
  country {
    countries_and_regions = [
      "GB",
      "US",
    ]
    include_unknown_countries_and_regions = false
  }
}
```

![Named Location Resource: IP Ranges](/assets/img/CABlog12.png)

![Named Location Countries](/assets/img/CABlog13.png)

 Once everything is in place and after a greenfield deployment this is how Conditional Access Policies will look like.

![Conditional Access Policies](/assets/img/CABlog15.png)

## **Ring Base Deployment**

One of the biggest takeaways from **Conditional** Access for Zero Trust Architecture** is ring-based deployment.

Let me try to summarize the concept of ring-based deployment. The idea is to deploy the Conditional Access Policies in a ring-based approach. The ring-based approach is based on the following principles.

### **Steps to deploy a new CA Policy**

- Create a new CA Policy within the persona group that it is meant for.
- Put it into a reporting-only mode to not affect production before testing.
- Let the policy run for a day or two and verify if end-users have complained about new login prompts. Potentially adjust the policy and continue running it in the report-only mode for a few extra days.
- Assign the policy to CA-Persona-Internals-Ring0 and enable it.
- Test and verify that everything is working as expected over a few days. Additionally, assign the policy to CA-Persona-Internals-Ring1 (so that both Ring0 and Ring1 groups are assigned).
- Test and verify that everything is working as expected over a few days.
- Additionally assign the policy to CA-Persona-Internals-Ring2 (so that both Ring0, Ring1, and Ring2 groups are assigned).
- Test and verify that everything is working as expected over a few days.
- Additionally assign the policy to CA-Persona-Internals-Ring3 (so that both Ring0, Ring1, and Ring2 groups are assigned).
- Test and verify that everything is working as expected over a few days.
- Assign the policy to CA-Persona-Internals. The new policy is now running in full production.

![Ring Based Deployment](/assets/img/CABlog16.png)

## **Use GitHub Actions for Automated Ring Base Deployment of new CA Policy**

### **Authentication Challenge of Terraform and GitHub Actions**

Our journey begins by addressing the crucial issue of authentication. For our purposes, we opt for the Client Credentials flow using Client ID and Client Secrets. However, securely managing these secrets for Terraform and subsequent **GitHub Workflow** integration poses a significant challenge for developers, DevOps teams, and IT administrators. While incorporating secrets into application configurations or exposing them as environment variables may appear intuitive, storing them directly in code and committing them to GitHub repositories can lead to potential security disasters. To counter this vulnerability, developers often seek secure storage solutions such as a vault. Nevertheless, this approach necessitates periodic expiration and rotation of secrets in Key Vaults, a task that, if not executed correctly, can introduce service availability challenges. Additionally, manual management of secrets carries the risk of individuals departing from the organization with access to critical secrets, potentially resulting in unauthorized resource access. Striking the right balance between security and service continuity becomes a complex endeavor in the realm of secrets management.

### **Advantages of Workload Identity Federation (WIF)**

We've chosen to implement Workload Identity Federation (WIF) due to its numerous advantages:

- **No Secrets to Manage**
- **Use of Azure AD Graph API Permissions**
- **Short-Lived Tokens**

So, how do federated identity credentials interact with GitHub and Entra ID?

The process begins by establishing a trust relationship between an external identity provider (IdP), in this case, **GitHub**, and an app in [Entra ID (by configuring)](#create-an-application-object-in-entra-id-tenant) with a federated identity credential. This federated identity credential specifies which tokens from GitHub should be trusted by your application. Once this trust relationship is established, GitHub can exchange trusted tokens from the external identity provider for access tokens from the Microsoft identity platform. GitHub then utilizes this access token to access Azure AD-protected resources granted access by the workload. This eliminates the manual credential management burden and minimizes the risk of secret leakage or certificate expiration. For additional information and supported scenarios, please refer to workload identity federation.



### **Create an Application Object in Entra ID Tenant**

Our initial step is to create an Application Object or **User Assigned Managed Identity** in Entra ID.

![FID App](/assets/img/FID2.jpg)

As a best practice, I recommend configuring this using the Graph API, facilitating easier migration between staging and production tenants.

```PowerShell
Import-Module Microsoft.Graph.Beta.Applications
Connect-MgGraph -TenantID '****************-2f34-4dc4-9500-****************' -Scope 'Application.ReadWrite.All'
$params = @{
 displayName = "GithubFederatedWIF2"
}

$Application = New-MgBetaApplication -BodyParameter $params

```

### **Add Federated Credential for GitHub Repo**


Follow the instructions mentioned in the steps below to add Federated Credentials for your Github Repository. As I am doing this using my Personal Github Repo, I went with the branch strategy.
![FID App](/assets/img/FID3.jpg)

![FID App](/assets/img/FID4.jpg)

![FID App](/assets/img/FID5.jpg)

```PowerShell
Import-Module Microsoft.Graph.Beta.Applications
$applicationId = $Application.Id


$params = @{
 name = "GithubFederatedWIFCredential"
 issuer = "https:/token.actions.githubusercontent.com"
 subject = "repo:SuryenduB/Terraform:ref:refs/heads/Master"
 audiences = @(
"api:/AzureADTokenExchange"
)
}

New-MgBetaApplicationFederatedIdentityCredential -ApplicationId $applicationId -BodyParameter $params
```

![FID App](/assets/img/FID6.jpg)

Given that our application's primary objective is to grant necessary permissions for modifying Conditional Access Policies, I'll add the following permissions to the Application Object `Policy.Read.All`, `Policy.ReadWrite.ConditionalAccess`, and `Application.Read.All`.

![FID App](/assets/img/FID7.jpg)

As we need to Obtain a token using Federated Credentials, we need to make the actions variable available as an Environment Variable.

```terraform
export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
export ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"
```

## Authenticating with Federated Credentials

To configure secrets or variables on GitHub, your level of access depends on the repository type:

1. **Personal Account Repository**: You must be the repository owner to create secrets or variables.

2. **Organization Repository**: Admin access is required to create secrets or variables for organization repositories.

3. **REST API Access**: To create secrets or variables for either personal accounts or organization repositories via the REST API, you need collaborator access.

Here's a step-by-step guide:

1. Go to GitHub.com and navigate to the main page of your repository.

2. Under the repository name, click on "Settings." If you don't see the "Settings" tab, click on the dropdown menu, then select "Settings."


3. In the sidebar, under the "Security" section, select "Secrets and variables," then click "Actions."

4. Click on the "Secrets" tab.

5. To create a new repository secret, click on "New repository secret."

![FID 9](/assets/img/FID9.1.jpg)

6. In the "Name" field, provide a name for your secret. In our case, we need to create two secrets: **AZURE_CLIENT_ID** and **AZURE_TENANT_ID**. Note that there's no need to store a client secret anymore.

7. In the "Secret" field, enter the value for your secret. For **AZURE_CLIENT_ID**, provide the Application ID of the application you've configured for federated credentials. For **AZURE_TENANT_ID**, provide the Tenant ID of the Entra ID Tenant.

8. Click "Add secret."

### Extend Terraform Folder for Conditional Access Policy Deployment

* Let us start by Creating a new branch to extend the existing ```azuread_conditional_access_policy.tf``` file to include the following code. We will be using the following code to generate the Conditional Access Policy for the Internal Persona and Deploy it for the Ring 0 group of the Internal Persona.

```hcl
resource "azuread_conditional_access_policy" "CA200-Internals-BaseProtection-AllApps-AnyPlatform-CompliantorAADHJ" {
  display_name = "CA200-Internals-BaseProtection-AllApps-AnyPlatform-CompliantorAADHJ"
  state        = "enabledForReportingButNotEnforced"
  conditions {
    applications {
      included_applications = ["All"]
      excluded_applications = [data.azuread_service_principal.intune.client_id]
    }
    platforms {
      included_platforms = ["all"]
    }
    users {
      included_groups = [azuread_group.CA-Persona-Rings["Internals.Ring0"].id]
      excluded_groups = [azuread_group.breakglass.id, azuread_group.CA-Persona-Groups-Exclusions["Internals.BaseProtection"].id]
    }
    client_app_types = ["all"]
  }
  grant_controls {
    operator          = "OR"
    built_in_controls = ["compliantDevice", "domainJoinedDevice"]
  }
}
```

Once we have the code ready we can commit the code to the Ring0 Branch and create a Pull Request. Once the Pull Request is created we can review and after reviewing the Pull Request we merge the test branch with the Main Branch.

### Workflow Trigger

This workflow is triggered in response to two events:

1. Pushes to the master branch.

```yaml
name: 'Terraform Plan/Apply'
on:
  push:
    branches:
    - main
```

#### 1. `terraform-plan`

This job focuses on the planning phase of the deployment process. It performs the following tasks: Check out the repository to access the latest code.
- Sets up the Terraform CLI.
- Changes the working directory to `EntraID/` where the Terraform configurations are stored.
- Initializes the Terraform working directory.
- Ensures that all Terraform configuration files adhere to the canonical format.
- Logs in to Azure CLI with Federated Credentials.
- Generates an execution plan for Terraform and saves it as `tfplan`.
- Publishes the Terraform plan as an artifact for later stages.

#### 2. `terraform-apply`

This job focuses on the application phase of the deployment process and is triggered under specific conditions:

- Only when changes are detected in the master branch (`github.ref == 'refs/heads/master'`).
- Only if the previous `terraform-plan` job resulted in pending changes (`needs.terraform-plan.outputs.tfplanExitCode == 2`).

The tasks within this job include:

- Checking out the repository.
- Setting up the Terraform CLI.
- Logging in to Azure CLI with Federated Credentials.
- Initializing the Terraform working directory.
- Downloading the saved Terraform plan (`tfplan` artifact) generated in the `terraform-plan` job.
- Applying the Terraform plan to the Azure environment.

This structured workflow ensures that the CA policy deployment process is orchestrated efficiently, with a clear separation between planning and application stages. Any changes detected in the master branch related to IAM configurations trigger the workflow, allowing for controlled and automated deployment.


```yaml
name: 'Terraform Plan/Apply'

on:
  push:
    branches:
    - master
    
    
  pull_request:
    branches:
    - master
    paths:

```

The job or workflow run requires a permissions setting with id-token: write. We won't be able to request the OIDC JWT ID token if the permissions setting for the id-token is set to read or none.

```yaml
#Special permissions required for OIDC authentication
permissions:
  id-token: write
  contents: read
  pull-requests: write
```

The ```terraform-plan```  job runs on an Ubuntu virtual machine in ***Github hosted runner*** following steps

```yaml

terraform-plan:
    name: 'Terraform Plan'
    runs-on: ubuntu-latest
    env:
      #this is needed since we are running terraform with read-only permissions
      ARM_SKIP_PROVIDER_REGISTRATION: true
    outputs:
      tfplanExitCode: ${{ steps.tf-plan.outputs.exitcode }}

```

1. Check out the repository to the GitHub Actions runner.

```yaml
steps:
    # Checkout the repository to the GitHub Actions runner
    - name: Checkout
      uses: actions/checkout@v3

```
2. Install the latest version of the Terraform CLI.

```yaml
# Install the latest version of the Terraform CLI
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_wrapper: false

```



3. Initializes the Terraform working directory by creating initial files, loading any remote state, and downloading modules.
```yaml
# Initialize a new or existing Terraform working directory by creating initial files, loading any remote state, downloading modules, etc from the EntraID Directory
    
    - name: Terraform Init
      run: terraform init
```

4. Format all Terraform configuration files to adhere to a canonical format.

```yaml
# Checks that all Terraform configuration files adhere to a canonical format
    # Will fail the build if not
    - name: Terraform Format
      run: terraform fmt -check
```

5. Logs in to Azure CLI with federated credentials.7. Generates an execution plan for Terraform and saves it to the tfplan file.

```yaml

# Generates an execution plan for Terraform
    # An exit code of 0 indicated no changes, 1 a terraform failure, 2 there are pending changes.
    - name: Terraform Plan
      id: tf-plan
      run: |
        export exitcode=0
        terraform plan -detailed-exitcode -no-color -out tfplan || export exitcode=$?

        echo "exitcode=$exitcode" >> $GITHUB_OUTPUT
        
        if [ $exitcode -eq 1 ]; then
          echo Terraform Plan Failed!
          exit 1
        else 
          exit 0
        fi
```

6. Publishes the Terraform plan as an artifact.

```yaml
# Save plan to artifacts  
    - name: Publish Terraform Plan
      uses: actions/upload-artifact@v3
      with:
        name: tfplan
        path: EntraID/tfplan

```

7. Creates a string output of the Terraform plan and publishes it as a task summary.

```yaml
# Create string output of Terraform Plan
    - name: Create String Output
      id: tf-plan-string
      run: |
        TERRAFORM_PLAN=$(  terraform show -no-color tfplan)
        
        delimiter="$(openssl rand -hex 8)"
        echo "summary<<${delimiter}" >> $GITHUB_OUTPUT
        echo "## Terraform Plan Output" >> $GITHUB_OUTPUT
        echo "<details><summary>Click to expand</summary>" >> $GITHUB_OUTPUT
        echo "" >> $GITHUB_OUTPUT
        echo '```terraform' >> $GITHUB_OUTPUT
        echo "$TERRAFORM_PLAN" >> $GITHUB_OUTPUT
        echo '```' >> $GITHUB_OUTPUT
        echo "</details>" >> $GITHUB_OUTPUT
        echo "${delimiter}" >> $GITHUB_OUTPUT
        
```

8. Post the Terraform plan as a comment on the pull request (if applicable).


```yaml
# If this is a PR post the changes
    - name: Push Terraform Output to PR
      if: github.ref != 'refs/heads/master'
      uses: actions/github-script@v6
      env:
        SUMMARY: "${{ steps.tf-plan-string.outputs.summary }}"
      with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const body = `${process.env.SUMMARY}`;
            github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: body
            })
```


Next, it is the turn of The ```terraform-apply``` job to run (similarly in  an Ubuntu virtual machine for Github Runner)  and perform the following steps:

```yaml
terraform-apply:
    name: 'Terraform Apply'
    env:
      #this is needed since we are running terraform with read-only permissions
      ARM_SKIP_PROVIDER_REGISTRATION: true
    
    if: github.ref == 'refs/heads/master' && needs.terraform-plan.outputs.tfplanExitCode == 2
    runs-on: ubuntu-latest
    needs: [terraform-plan]
```

- Check out the repository to the GitHub Actions Runner.

```yaml

    # Checkout the repository to the GitHub Actions runner
    - name: Checkout
      uses: actions/checkout@v3

```

- Installs the latest version of the Terraform CLI and configures the Terraform CLI configuration file with a Terraform Cloud user API token.

```yaml
# Install the latest version of Terraform CLI and configure the Terraform CLI configuration file with a Terraform Cloud user API token
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
```

- Log in to Azure CLI with federated credentials.

- Initializes the Terraform working directory by creating initial files, loading any remote state, and downloading modules.

- Download the saved Terraform plan from the tfplan artifact from the previous Job.

```yaml

- name: Terraform Init
      run:  terraform init

# Download saved plan from artifacts  q
- name: Download Terraform Plan
 uses: actions/download-artifact@v3
      with:
        name: tfplan
```

- Applies the Terraform plan to the Azure environment.

```yaml
# Terraform Apply
    - name: Terraform Apply
      run:  terraform apply -auto-approve ./tfplan

```

![Workflow Triggered](/assets/img/FID9.3.jpg)

We can see the Workflow Steps for Terraform Plan and Terraform Apply have been completed.

![Workflow Triggered](/assets/img/FID9.4.jpg)

If we look at each step of ```terraform-plan``` job we can see all the steps are completed.

![Workflow Triggered](/assets/img/FID9.5.jpg)

Similarly, for the ```terraform-apply``` job, we can verify all the necessary steps are completed.

![Workflow Triggered](/assets/img/FID9.6.jpg)

We can also verify the Conditional Access Policy is deployed in the Entra ID Portal.

Once the first deployment is successful we can repeat the steps to run the workflow for the next Ring. In our case, we will be updating the policy for Ring 1.

```hcl
 users {
      included_groups = [azuread_group.CA-Persona-Rings["Internals.Ring0"].id, azuread_group.CA-Persona-Rings["Internals.Ring1"].id
 }
      
```

and gradually to all the Rings and finally to complete Persona.

```hcl
included_groups = [azuread_group.CA-Persona-Rings["Internals.Ring0"].id, azuread_group.CA-Persona-Rings["Internals.Ring1"].id, azuread_group.CA-Persona-Rings["Internals.Ring2"].id, azuread_group.CA-Persona-Rings["Internals.Ring3"].id,azuread_group.CA-Persona-Groups["Internals"].id]
```

- Let the policy run for a day or two, - based on potential failures in CA workbooks and sign-in logs assuring they are understood before proceeding and verify if end-users have complained about new login prompts i.e. especially on mobile devices as report-only can result in unexpected prompts in a few use-cases.

- Potentially adjust the policy and continue running it in the report-only mode for a few extra days and verify that issues have been solved or fully understood before enabling the policy for the first ring.

- Assign the policy to CA-Persona-Internals-Ring0 and enable it.

```hcl
 state = enabled
 
 users {
      included_groups = [azuread_group.CA-Persona-Rings["Internals.Ring0"].id]
 }
      
```

- Test and verify that everything is working as expected over a few days

- Additionally, assign the policy to CA-Persona-Internals-Ring1 (so that both Ring0 and Ring1 groups are assigned)
- Test and verify that everything is working as expected over a few days
- Additionally, assign the policy to CA-Persona-Internals-Ring2 (so that both Ring0, Ring1, and Ring2 groups are assigned)

- Test and verify that everything is working as expected over a few days
- Additionally assign the policy to CA-Persona-Internals-Ring3 (so that both Ring0, Ring1, Ring2, and Ring3 are assigned.

- Finally assigned the policy to CA-Persona-Internals. The new policy is now running in full production.

```hcl
users {
      included_groups = [azuread_group.CA-Persona-Groups["Internals"].id]
     
```

![CA-Policy](/assets/img/CABlog17.png)

## Final Thoughts

 **The Conditional Access For Zero Trust** framework is a powerful framework that aligns with **Modern Security Best Practices** to secure the entire enterprise. This blog shows how any Organization can securely manage and deploy Conditional Access Policies **Policies-As-A-Code** using Terraform and **GitHub Actions**. In this PoC Example, GitHub Repo is the **Source of Truth** that is maintained using the **branch protection rule**, and **code review** for merging the feature branch to the master Branch. CI/CD Pipeline is triggered whenever there is a new update to the Master Branch. Refer to the reference section for [Conditional Access for Zero Trust Architecture](https:/github.com/microsoft/ConditionalAccessforZeroTrustResources/blob/main/ConditionalAccessGovernanceAndPrinciplesforZeroTrust%20October%202023.pdf) as the starting point.

## References

- [Conditional Access for Zero Trust Architecture](https:/github.com/microsoft/ConditionalAccessforZeroTrustResources/blob/main/ConditionalAccessGovernanceAndPrinciplesforZeroTrust%20October%202023.pdf)
- [Hashicorp Azure AD Module](https:/registry.terraform.io/providers/hashicorp/azuread/latest/docs)
- [Workload Identity Federation](https:/learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation)
- [AADOps: Operationalization of Azure AD Conditional Access](https://www.cloud-architekt.net/aadops-conditional-access/#definition-of-requirement-and-plan-deployment)

---
layout: post
title: Simplifying GitHub Authentication with Entra ID Federated Credentials- Terraforming Conditional Access Policies via GitHub Workflows - Part 1
subtitle :   Unlocking Secure and Automated Deployment of Conditional Access Policy
cover-img: /assets/img/FID1.jpg
thumbnail-img: /assets/img/FID1.jpg
share-img: /assets/img/FID1.jpg
tags: [ Azure Active Directory, EntraID, MicrosoftEntra, Security, ZeroTrust, Terraform, Devops]

---
# Simplifying GitHub Authentication with Entra ID Federated Credentials: Terraforming Conditional Access Policies via GitHub Workflows - Part 1

- [Simplifying GitHub Authentication with Entra ID Federated Credentials: Terraforming Conditional Access Policies via GitHub Workflows - Part 1](#simplifying-github-authentication-with-entra-id-federated-credentials-terraforming-conditional-access-policies-via-github-workflows---part-1)
  - [Introduction](#introduction)
  - [Steps](#steps)
    - [Authentication Challenge](#authentication-challenge)
    - [Advantages of Workload Identity Federation (WIF)](#advantages-of-workload-identity-federation-wif)
    - [Create an Application Object in Entra ID Tenant](#create-an-application-object-in-entra-id-tenant)
    - [Add Federated Credential for GitHub Repo](#add-federated-credential-for-github-repo)
  - [Conclusion](#conclusion)

## **Introduction**

Several years ago, I found myself facing a captivating challenge: deploying Privilege Identity Management (PIM) roles using Terraform and orchestrating the deployment process through GitHub Workflows. The organization I worked with was no stranger to the cloud, hosting its entire infrastructure there and wholeheartedly embracing automated continuous deployment as the standard practice for all deployment operations. Their unwavering commitment to this approach extended even to Identity and Access Management (IAM) deployment, a decision that, as you'll discover, was entirely justified.

My background primarily revolved around IAM intricacies, and this project propelled me into the realm of DevOps – a world characterized by automation and unceasing workflows. I was, to be honest, a relative newcomer to this landscape. Yet, this project was not just another technical endeavor; it was an opportunity to bridge the gap between IAM and DevOps, leveraging GitHub authentication, Azure Active Directory (Azure AD), and Terraform.

This journey was a challenge worth embracing, a path that would reveal the remarkable potential of blending IAM and DevOps in a more agile and efficient manner. In this article, we'll take a technical dive into the process I followed, detailing the authentication flow within Entra ID, and the deployment of conditional access policies using Terraform – all orchestrated through the magic of GitHub Workflows. Join us as we explore how this combination of technologies can simplify your access management and enhance security within the DevOps realm.


### **Authentication Challenge**

Our journey begins by addressing the crucial issue of authentication. For our purposes, we opt for the Client Credentials flow using Client ID and Client Secrets. However, securely managing these secrets for Terraform and subsequent **GitHub Workflow** integration poses a significant challenge for developers, DevOps teams, and IT administrators. While incorporating secrets into application configurations or exposing them as environment variables may appear intuitive, storing them directly in code and committing them to GitHub repositories can lead to potential security disasters. To counter this vulnerability, developers often seek secure storage solutions such as a vault. Nevertheless, this approach necessitates periodic expiration and rotation of secrets in Key Vaults, a task that, if not executed correctly, can introduce service availability challenges. Additionally, manual management of secrets carries the risk of individuals departing from the organization with access to critical secrets, potentially resulting in unauthorized resource access. Striking the right balance between security and service continuity becomes a complex endeavor in the realm of secrets management.

### **Advantages of Workload Identity Federation (WIF)**

We've chosen to implement Workload Identity Federation (WIF) due to its numerous advantages:

- **No Secrets to Manage**
- **Use of Azure AD Graph API Permissions**
- **Short-Lived Tokens**

So, how do federated identity credentials interact with GitHub and Entra ID?

The process begins by establishing a trust relationship between an external identity provider (IdP), in this case, **GitHub**, and an app in [Entra ID (by configuring)](#create-an-application-object-in-entraid-tenant) with a federated identity credential. This federated identity credential specifies which tokens from GitHub should be trusted by your application. Once this trust relationship is established, GitHub can exchange trusted tokens from the external identity provider for access tokens from the Microsoft identity platform. GitHub then utilizes this access token to access Azure AD-protected resources granted access by the workload. This eliminates the manual credential management burden and minimizes the risk of secret leakage or certificate expiration. For additional information and supported scenarios, please refer to workload identity federation.

## Steps

![FID App](/assets/img/FID1.jpg)

### **Create an Application Object in Entra ID Tenant**

Our initial step is to create an Application Object or **User Assigned Managed Identity** in Entra ID.

![FID App](/assets/img/FID2.jpg)

As a best practice, I recommend configuring this using the Graph API, facilitating easier migration between staging and production tenants.

```powershell
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

```powershell
Import-Module Microsoft.Graph.Beta.Applications
$applicationId = $Application.Id


$params = @{
 name = "GithubFederatedWIFCredential"
 issuer = "https://token.actions.githubusercontent.com"
 subject = "repo:SuryenduB/Terraform:ref:refs/heads/Master"
 audiences = @(
"api://AzureADTokenExchange"
)
}

New-MgBetaApplicationFederatedIdentityCredential -ApplicationId $applicationId -BodyParameter $params
```

![FID App](/assets/img/FID6.jpg)

Given that our application's primary objective is to grant necessary permissions for modifying Conditional Access Policies, I'll add the following permissions to the Application Object Policy.Read.All, Policy.ReadWrite.ConditionalAccess, and Application.Read.All.

![FID App](/assets/img/FID7.jpg)

To maintain brevity, I'll conclude here but offer a glimpse of what to expect in subsequent parts of this article:

1. Importing existing Conditional Access Policies using Terraform 1.5 Import Module.
2. Adapting the generated Terraform Code to align with our requirements.
3. Creating a GitHub Workflow in GitHub Hosted Runner to deploy the new Conditional Access Policy.

All of this, and more, is accomplished without relying on cloud secrets.

## **Conclusion**


In this article, we have discussed how to use Entra ID federated credentials to authenticate to Azure AD and deploy conditional access policies using Terraform and GitHub Workflows. This approach offers a number of benefits, including:

* **Eliminates the need to manage secrets:** With Entra ID federated credentials, there is no need to store or manage secrets in your code or environment variables. This reduces the risk of unauthorized access and makes your code more secure.
* **Simplifies deployment:** By using GitHub Workflows to deploy your conditional access policies, you can automate the process and ensure that your policies are always up-to-date. This can save you time and effort, and help you to improve your overall security posture.

In the next part of this series, we will show you how to import existing conditional access policies using Terraform and make the necessary changes to the generated Terraform code to reflect your changes. We will also show you how to create a GitHub workflow to deploy the new conditional access policy.

We hope you have found this article helpful. Please stay tuned for the next part of the series!
[Cover Image from IdentityDigest blog.](https://blog.identitydigest.com/azuread-federate-github-actions/)

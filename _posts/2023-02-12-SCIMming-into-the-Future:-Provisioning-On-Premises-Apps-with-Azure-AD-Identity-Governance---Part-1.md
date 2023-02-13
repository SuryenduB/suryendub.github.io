---
layout: post
title: SCIMming into the Future - Provisioning On-Premises Apps with Azure AD Identity Governance - Part 1
subtitle: Automate provisioning and governance of your on-premises applications
cover-img: /assets/img/SCIM-Identity-Governance.jpg
thumbnail-img:  /assets/img/SCIM-Identity-Governance.jpg
share-img: /assets/img/SCIM-Identity-Governance.jpg
tags: [AzureAD,IdentityGovernance,SCIMProvisioning,OnPremisesApplications,MicrosoftEntra,UserManagement,HybridEnvironment]
---



# SCIMming into the Future: Provisioning On-Premises Apps with Azure AD Identity Governance - Part 1

## Introduction: A brief overview of the specific Azure AD functionality being discussed.

![Image for SCIM Provisioning](/assets/img/SCIM-Identity-Governance.jpg)

One of the major challenges in widespread adoption of Azure Active Directory (AD) has been the limited support for managing users and groups in on-premises applications. Although there has been an increase in cloud-only organizations, the majority of organizations still operate in a hybrid environment and rely on on-premises Line of Business (LOB) applications for managing their enterprises. While system administrators may eventually hope for a future where all enterprises are 100% in the cloud, it is important to recognize that this is still a work in progress. 

The latest offering from Microsoft is a significant step forward in the right direction. With the now generally available functionality of provisioning to on-premises applications using [Microsoft Entra Identity Governance](https://learn.microsoft.com/en-us/azure/active-directory/governance/identity-governance-overview) for on-premises applications, IAM engineers have a powerful tool for designing and implementing user and group lifecycle management for on-premises applications.

The benefits of this new offering are numerous.
 1. Organizations can now reduce their dependence on complex identity management solutions like Sailpoint and Saviynt, freeing up IT budget for other initiatives. Additionally, custom solutions that are based on PowerShell scripts can be unreliable and often create security blind spots due to a lack of documentation and source code management. With Microsoft Entra Identity Governance, identity provisioning and governance can be effectively managed with a no-code solution.
 2. Latest offering from Microsoft, is a path in the right direction. Now with generally available functionality  of provisioning to on-premises applications using [Microsoft Entra Identity Governance](https://learn.microsoft.com/en-us/azure/active-directory/governance/identity-governance-overview) IAM engineers can design us lifecycle management of users and group for  on-premises applications.



## Use Cases: A discussion of the common use cases for the specific functionality, including real-world examples or scenarios.

Here are few usecases,

1. **Integration with HR Systems:** SCIM Provisioning with Azure AD Identity Governance can be integrated with HR systems, such as Workday or SuccessFactors, to automatically provision and de-provision user accounts in on-premises applications based on changes made in the HR system.
2. **Automated User Onboarding:** SCIM Provisioning with Azure AD Identity Governance quickly onboard new employees and grant them access to the on-premises applications and data they need to perform their job functions, without the need for manual provisioning and access requests.
3. **Security and Compliance:**  It can help organizations identify and eliminate access rights that are no longer needed or are assigned to users who are no longer employed. This helps minimize the risk of data breaches and unauthorized access to on-premises applications.

In this multi-part blog series, we shall delve into the following aspects: 
1. Microsoft offers a [test application](https://aka.ms/scimreferencecode) that enables you to construct your own System for Cross-domain Identity Management (SCIM) endpoint. We shall utilize this application to construct our own SCIM endpoint. 

2. We will host the application in Azure App Services.

3. We will test the application using Postman.

4. We will test the application with Azure AD On-demand Application provisioning.

5. We will Create Access Package and utomatic assignment policy to assign the application to new hires (provisioning).

6. Create multi-staged access review to remove access for users who no longer require access to the application.


Microsoft Entra Identity Governance for on-premises applications provides a powerful tool for IAM engineers to design and implement user and group lifecycle management.

We will end the first part here. Stay tuned for the next post.

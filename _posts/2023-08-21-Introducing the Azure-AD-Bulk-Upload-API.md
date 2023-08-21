---
layout: post
title: Introducing the Azure AD Bulk Upload API, The Next Generation of Provisioning for Enterprise IAM

cover-img: /assets/img/Existing%20Identity%20Enterprise%20.jpg
thumbnail-img: /assets/img/Future%20Identity%20Enterprise%20Bulk%20API%20.jpg
share-img: /assets/img/Future%20Identity%20Enterprise%20Bulk%20API%20.jpg
tags: [ Azure Active Directory, EntraID , AzureAD , MicrosoftGraph , Provisioning , BulkAPI , MIM]

---
# Replacing MIM: Introducing the Azure AD Bulk Upload API, The Next Generation of Provisioning for Enterprise IAM

In a manner reminiscent of the content presented on this blog "MIM and Beyond," each successive release from Microsoft, n with the introduction of new Entra ID releases, demonstrates a gradual and purposeful shift towards more effectively replacing Microsoft Identity Manager (MIM).

Within enterprise environments, Microsoft Identity Manager (MIM) offers a distinctive capability by virtue of its comprehensive integration prowess. This is exemplified through the utilization of the [Extensible Connectivity 2.0 Management Agent Reference](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/bb891983(v=vs.100) "ECMA"), the [Windows PowerShell Connector](https://learn.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-powershell "Optional title"), and the [Grendfelt Powershell Management Agent](https://github.com/sorengranfeldt/psma "psma"). These mechanisms enable seamless connections with various systems encompassing the enterprise landscape.

Furthermore, the intrinsic capability to tailor data to align with the distinct requirements of enterprises is a key facet of MIM's value proposition. This customization is facilitated through tools such as [MIMWAL](https://microsoft.github.io/MIMWAL/) and the [Management Agent Rules Extension](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ms698793(v=vs.100)), enabling efficient data transformation processes.

In view of the aforementioned features and capabilities, Microsoft Identity Manager (MIM) emerges as a highly useful utility within the enterprise context.

![Placeholder for Image For Existing Enterprise Architecture](/assets/img/Existing%20Identity%20Enterprise%20.jpg)

However, Microsoft Entra ID has come up with capabilities for matching MIM Capabilities as follows.

| Identity Capability | Solution using MIM | Solution using Entra ID |
| -------- | -------- | -------- |
| Integration with well known HR Data Solution | [Microsoft Identity Manager PowerShell Management Agent for Workday HR](https://blog.darrenjrobinson.com/building-a-microsoft-identity-manager-powershell-management-agent-for-workday-hr/ "Optional title") , | [HR application to Azure Active Directory user provisioning](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/what-is-hr-driven-provisioning#cloud-hr-application-to-azure-active-directory-user-provisioning) |
| <span style="color:red"> Integration with In House HR Data Solution  | <span style="color:red"> [File-Based ECMA 2.2 Management Agent](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859565(v=vs.100))| <span style="color:red">  [API-driven inbound provisioning](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/inbound-provisioning-api-concepts#scenario-1-enable-it-teams-to-import-hr-data-extracts-using-any-automation-tool) |
| Provision to SAAS Application | [Windows PowerShell Connector](https://github.com/microsoft/MIMPowerShellConnectors/tree/69963a1b7ab263fcf33662a3b643766beeb65eab) | [App provisioning in Azure Active Directory](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/user-provisioning) |
| Provision to On-Premise Application | [MIM Connector Reference](https://learn.microsoft.com/en-us/microsoft-identity-manager/supported-management-agents)| [Azure AD on-premises application identity provisioning](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/on-premises-application-provisioning-architecture) |
| Provision to On-Premise Active Directory | [How Do I Provision Users to AD DS](https://learn.microsoft.com/en-us/microsoft-identity-manager/mim-how-provision-users-adds) | [Cloud HR app to Active Directory user provisioning](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/plan-cloud-hr-provision#select-cloud-hr-provisioning-connector-apps) |
|  Custom Transformation in Provisioning  |  [Transforming Data Using Extension Code](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ms698793(v=vs.100)) |  [Understand how expression builder in Application Provisioning works](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/functions-for-customizing-application-data) |

## Introducing the Azure AD Bulk Upload API: The Future of Provisioning 🚀

The Azure AD provisioning service just got a major upgrade making us one step closer towards MIM Replacement and even Active Directory Replacement Now, it plays super well with all sorts of HR systems. 🎉 Enterprise IAM Admins  can take their pick of automation tools ( [Powershell](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/inbound-provisioning-api-powershell) or [Logic App](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/inbound-provisioning-api-logic-apps) ) to integrate employee data from their record systems and smoothly transfer it into Azure AD or On Premises AD.

 🌟 Once your workforce data is in Azure AD, you can use Lifecycle Workflows to automate onboarding, offboarding, and other common identity lifecycle Joiner , Mover , Leaver tasks.  📊💼

![Placeholder for Image For New Enterprise Architecture](/assets/img/Future%20Identity%20Enterprise%20Bulk%20API%20.jpg)


## Describe how to convert normal user to Cloud Only User

## Describe once the connection is broken how you can leverage Bulk API to update the user

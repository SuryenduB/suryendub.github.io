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


## Steps for Removing Active Directory from Identity Lifecycle Process

### The Big Bang Approach

This method involves turning off directory sync, converting users to Cloud Only Users, and uninstalling Azure AD Connect. A key part of this approach is connecting HR data with Entra ID seamlessly.

```Powershell

# Connect to the Microsoft Graph API with Organization.ReadWrite.All permission scope
Connect-MgGraph -scopes Organization.ReadWrite.All

# Check if the Microsoft.Graph.Beta.Identity.DirectoryManagement module is installed, and install it if necessary
$module = Get-Module "Microsoft.Graph.Beta.Identity.DirectoryManagement" -ListAvailable
if($null -eq $module)
{
    Install-Module Microsoft.Graph.Beta.Identity.DirectoryManagement -scope currentuser -force
}

# Get the ID of the current organization
$OrgID = (Get-MgOrganization).id

# Define parameters to update the organization's on-premises sync settings
$params = @{
 onPremisesSyncEnabled = $null
}

# Update the organization's on-premises sync settings using the Microsoft.Graph.Beta.Identity.DirectoryManagement module
Update-MgBetaOrganization -OrganizationId $OrgID -BodyParameter $params

```

You can verify the  value of the OnPremesisSyncEnabled property, you can use the Select statement as follows:

```powershell
Get-MgOrganization | Select OnPremisesSyncEnabled
```

However, this approach makes me nervous for the following reason.

1. The success of the Big Bang Approach depends on how well HR data and Entra ID integrate. Keep in mind that this integration is still in the early stages, so be cautious when rolling it out across your organization. Make sure the integration is solid before taking this big step. Building a complex integration process that matches all the requirements of an enterprise takes considerable skills and iterative development phases.

2. Once you disable sync for the whole organization, reversing users back to on-prem managed hybrid users gets tricky. So, think things through before jumping into this major change.

### A Thoughtful Transition: Phased Conversion from Hybrid Users to Cloud Users

A phased approach to convert hybrid users into cloud users can offer a prudent and controlled transition.

**1. Assessment and Planning 📊

We select a small batch of users for gradual migration. Develop a clear plan outlining the migration timeline, potential challenges, and mitigation strategies.

**2. Pilot Phase 🛫

- Temporarily stop identity synchronization. This will prevent Azure AD Connect from updating the cloud-based identity with any changes made to the on-premises identity.

```PowerShell
    Set-ADSyncScheduler -SyncCycleEnabled $false
```

- Move the designated users outside the scope of Azure AD Connect. This can be done by creating a new, separate OU in Active Directory (AD) and moving the users to that OU.

```PowerShell
# Connect to Active Directory
Import-Module ActiveDirectory

# Get the distinguished names (DNs) of the users to move
$user1DN = (Get-ADUser -Identity User1).DistinguishedName
$user2DN = (Get-ADUser -Identity User2).DistinguishedName

# Move the users to the new OU
Move-ADObject -Identity $user1DN -TargetPath "OU=NonCloudSynced,DC=example,DC=com"
Move-ADObject -Identity $user2DN -TargetPath "OU=NonCloudSynced,DC=example,DC=com"
```

**3.Resume synchronization and force a delta sync. This will synchronize the cloud-based identities with the on-premises identities, but only for the users that have been moved to the new OU.

```PowerShell
Set-ADSyncScheduler -SyncCycleEnabled $true
Start-ADSyncSyncCycle -PolicyType Delta
```

**4. Restore users from the recycle bin in Azure AD. These users are now orphaned, so they need to be restored before they can be used.
**5. Fix any attributes and data for the 'new' users. This may include updating their passwords, email addresses, or other attributes.

```powershell
$OldUPN = "firstname.lastname@example.com"
$cloudTenant = "onprem.onmicrosoft.com"
$UserName = ($OldUPN -split '@')[0]
$oldSuffix = ($OldUPN -split '@')[1]
$tempUPN = "${UserName}@${cloudTenant}"
$NewUPN = $OldUPN 


$filterUrl = "https://graph.microsoft.com/v1.0/directory/deletedItems/microsoft.graph.user?\$filter=endsWith(userPrincipalName,'$OldUPN')&\$count=true"

$deletedObj = Invoke-MgGraphRequest -Uri $filterUrl -Method Get -ErrorAction Stop -Headers @{ConsistencyLevel = 'eventual' }

if ($deletedObj.'@odata.count' -eq 0) {
    # Output error or do nothing
    Write-Error "User $OldUPN not found in deleted items"
}
else {
    $userId = $deletedObj.Value[0].id
    Write-Output $userId

    Restore-MgDirectoryDeletedItem -DirectoryObjectId $userId | Out-Null
    Invoke-MgGraphRequest -Uri "https://graph.microsoft.com/v1.0/users/$userId" -Method PATCH -Body @{ OnPremisesImmutableId = $null } -Debug
    Start-Sleep -Seconds 10
    Update-MgUser -UserId $OldUPN -OnPremisesImmutableId " "
    Start-Sleep -Seconds 5
    $userObj = Get-MgUser -UserId $NewUPN -Property OnPremisesImmutableId
    if ($null -eq $userObj.OnPremisesImmutableId) {
        "All good!"
    }
}
```

The original AD identities cannot be moved back to their initial structure, as they would be matched again with the cloud-based identities. Ideally, the on-premises AD account would be disabled or permanently deleted after a grace period. It is sometimes necessary to tweak the immutableId, but this is not strictly required.


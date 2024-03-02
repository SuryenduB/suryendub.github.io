---
layout: post
title: Powershell Connector for EntraID
subtittle: Integrate with All Types of Applications with Powershell Connector
cover-img: /assets/img/RoleActivationLogo.png.jpg
thumbnail-img: /assets/img/RoleActivationThumbnail.png
share-img: /assets/img/RoleActivationThumbnail.png
tags: [ IAM, IGA , ENTRA, ENTRAID , ConditionAccess , CA Policy]
readtime: true

---
#  Powershell Connector for EntraID

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Entra ID PowerShell Connector](#entra-id-powershell-connector)
- [Setup On-Premises Agent](#setup-on-premises-agent)
- [Setup Powershell Connector](#setup-powershell-connector)
- [Setting Up Entra ID On-Premises Application](#setting-up-entra-id-on-premises-application)
- [Testing the Connector](#testing-the-connector)
- [Scripts](#scripts)
  - [Import Script](#import-script)
  - [Export Script](#export-script)


## Introduction

In today's technological landscape, many organizations still rely on legacy systems and applications. For small and medium-sized businesses, the cost of upgrading or replacing these systems with advanced cloud-based solutions can be a challenge. Additionally, these legacy applications often do not meet the SCIM standard for REST API integrations, making it impractical to build a SCIM framework for them.

As an IAM consultant, I frequently encounter the need to integrate legacy applications with cloud-based IAM solutions. This integration is crucial for organizations to leverage the benefits of cloud-based IAM while seamlessly connecting with their existing systems. To address this challenge, the Entra ID PowerShell Connector offers a powerful tool for IAM professionals familiar with Microsoft Identity Manager.

I recently came across this [blog post](https://learn.microsoft.com/en-us/microsoft-identity-manager/migrate-entra-id) by inimitable [Mark Wahl](https://www.linkedin.com/in/mawahl/) on how to Migrate from Microsoft Identity Manager to Entra ID. Over my limited experience with working with IT professionals, I have found many of the features are widely in use and organizations are greatly benefitting from it. However, non-MIM Entra ID engineers are hesitant to use some of the on-premises integration tools. This blog post is an attempt to bridge that gap and provide a simple and easy to use PowerShell connector for Entra ID to integrate with legacy applications.

I do not intend to make it a deep dive into the Entra ID PowerShell Connector, but rather a simple and easy-to-use guide for IT professionals to get started with it. I will be covering the following topics in this blog post:

- Download the Sample Connector from GitHub
- Update the `Schema.XML` file to match user attributes in your target application.
- Update the `Import.ps1` file to Import user data from your target application to Entra ID Provisioning Service.
- Update the `Export.ps1` file to Export user data from Entra ID Provisioning Service to your target application.
- Modify the Common Module Script to add Custom Functions.

## Prerequisites

## Entra ID PowerShell Connector

The Entra ID PowerShell Connector is a component of the Entra ID On-Premises Integration Tools. It is an implementation of the **Extensible Connectivity 2.2 Management Agent** using PowerShell, provided by Microsoft. This connector enables you to connect to various systems and applications for seamless integration.

![Entra ID Powershell Connector](/assets/img/PowershellConnectorDiagram.png)

## Setup On-Premises Agent

The first thing we need is to establish a connection between our on-premises environment and Entra ID. We will need to install the [cloud sync agent](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/reference-version-history) on the server we want to run our Powershell Agent.

 1. In the Azure portal, select **Microsoft Entra ID**.
 2. On the left, select **Microsoft Entra Connect**.
 3. On the left, select **Cloud sync**.
 4. On the left, select **Agent**.
 5. Select **Download on-premises agent**, and select **Accept terms & download**.
 6. Once the **Microsoft Entra Connect Provisioning Agent Package** has completed downloading, run the *AADConnectProvisioningAgentSetup.exe* installation file from your downloads folder.
 7. On the splash screen, select **I agree to the license and conditions**, and then select **Install**.
 8. Once the installation operation is completed, the configuration wizard will launch. Select **Next** to start the configuration.
 9. On the **Select Extension** screen, select **On-premises application provisioning (Microsoft Entra ID to application).** and click **Next**.
 10. Sign in with your Microsoft Entra Global Administrator or Hybrid Identity Administrator account.  If you have Internet Explorer enhanced security enabled, it will block the sign-in.  If so, close the installation, [disable Internet Explorer enhanced security](/troubleshoot/developer/browsers/security-privacy/enhanced-security-configuration-faq), and restart the **Microsoft Entra Connect Provisioning Agent Package** installation.

<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorOnPremisesAgentSetup.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

Once the installation is completed, you will see the **Microsoft Azure AD Connect Provisioning Agent Package** in the list of installed programs. Yes, this is one of the few areas where you will not see the Entra ID branding yet.

![Provisioning Agent Installed](/assets/img/PowershellConnectorInstalled.png)

We can verify the agent is now showing up in the Entra ID Portal.

1. Log in to the [Microsoft Entra admin center](https://entra.microsoft.com).
2. Select Microsoft Entra Connect, and then select Cloud sync.
3. On the cloud sync page, you'll see the agents you've installed. Verify that the agent is displayed and the status is healthy.

![Provisioning Agent Verified in Entra ID](/assets/img/PowershellConnectorAgentInPortal.png)

## Setup Powershell Connector

Before setting up the Powershell connector, you need to have the following files updated and placed in the correct directory. We will not need to modify the InputFile.txt file for our use case.
However, we need to modify the Schema.xml file.


| File         | Location                                      |
|--------------|-----------------------------------------------|
| InputFile.txt| C:\Program Files\Microsoft ECMA2Host\Service\ECMA\MAData |
| Schema.xml   | C:\Program Files\Microsoft ECMA2Host\Service\ECMA\MAData |

Our target application is a REST API application that does not have a SCIM standard. For our use case, we need to  be using the following attributes:

| Attribute Name | Description |
|----------------|-------------|
| login          | User's login |
| firstName      | User's first name |
| lastName       | User's last name |
| nickName       | User's nickname |
| displayName    | User's display name |
| email          | User's email address |
| secondEmail    | User's secondary email address |
| profileUrl     | User's profile URL |
| preferredLanguage | User's preferred language |
| userType       | User's type |
| organization   | User's organization |
| title          | User's title |
| division       | User's division |
| department     | User's department |
| costCenter     | User's cost center |
| employeeNumber | User's employee number |
| mobilePhone    | User's mobile phone number |
| primaryPhone   | User's primary phone number |
| streetAddress  | User's street address |
| city           | User's city |
| state          | User's state |
| zipCode        | User's zip code |
| countryCode    | User's country code |
| status| User's status : Active or Inactive |

For our Target Application, the attribute `login` is the unique identifier for the user. We will use this attribute to uniquely identify the user in the target application. 

Hence in our modified Schema.xml file, we will have the attribute as the anchor attribute and the other attributes as the flow attributes. The Schema.xml file will look like this:

```xml
<SchemaAttribute>
          <Name>login</Name>
          <DataType>String</DataType>
          <IsAnchor>1</IsAnchor>
          <IsMultiValued>0</IsMultiValued>
          <AllowedAttributeOperation>ImportExport</AllowedAttributeOperation>
</SchemaAttribute>
```

Pay special attention to the `IsAnchor` attribute. This is the attribute that is going to be used as the anchor attribute in ECMA2 host.
I promised not to do a deep-dive but it is important to understand how the schema is getting populated. Open the common module script and look for the `ConvertFrom-SchemaXml` function.  This will read the attribute for the schema.xml file and populate the schema for the ECMA2 host. `IsAnchor` is used to add the anchor attribute to the schema.

```powershell
  $schemaType = [Microsoft.MetadirectoryServices.SchemaType]::Create($t.Name,$lockAnchorDefinition)
  ...
```


```powershell
  $schemaType = [Microsoft.MetadirectoryServices.SchemaType]::Create($t.Name,$lockAnchorDefinition)
  ...
foreach ($a in $t.Attributes.SchemaAttribute)
      {
        if ($a.IsAnchor -eq 1)
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateAnchorAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
        elseif ($a.IsMultiValued -eq 1)
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateMultiValuedAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
        else
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateSingleValuedAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
      }


```
Configure the Microsoft Entra ECMA Connector Host certificate

1. On the Windows Server where you have installed the provisioning agent is installed, right click the **Microsoft ECMA2Host Configuration Wizard** from the start menu, and run as administrator. Running as a Windows administrator is necessary for the wizard to create the necessary Windows event logs.
2. After the ECMA Connector Host Configuration starts, if it's the first time you have run the wizard, it will ask you to create a certificate. Leave the default port 8585 and select Generate certificate to generate a certificate. The autogenerated certificate will be self-signed as part of the trusted root. The certificate SAN matches the hostname.
Select Save.

![ECMA Connector Host Configuration](/assets/img/PowershellConnectorGenerateCertificate.png)

 This is for securing communication between Entra ID and the agent.

 After Generating the certificate, you will see an empty list. This is where you can either start by clicking on **+New Connector** or **Import** the connector from my GitHub repository.

 For this blog post as a step-by-step guide, I will select the first option.

### General Page

 I have used the following values for the connector.

   |Property|Value|
   |-----|-----|
   |Name|PowershellRestAPI|
   |Autosync timer (minutes)|120|
   |Secret Token|You will need this token in the next step.|
   |Extension DLL|For the PowerShell connector, select **Microsoft.IAM.Connector.PowerShell.dll**.|


<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorPropertiesPage.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

### Connectivity  Page

On the connectivity Page, I have used the following attributes:

|Parameter|Value|Purpose|
|----|-----|-----|
|  Server  | \<Blank\> | Server name that the connector should connect to. For our configuration, this can remain blank.  |
|  Domain  | \<Blank\> |Domain of the credential to store for use when the connector is run. For our configuration, this can remain blank.|
|User| APIKey |  Username of the credential to store for use when the connector is run. I have used it to denote that the Credentials I am going to use is an **APIKey** we will use to connect.  |
| Password | Value of the API Key |  Password of the credential to store for use when the connector is run. I have used this to store the **APIKey** for connecting with the REST API. In that way we can ensure that credentials and API keys are not stored in plain text. I have used a function `GetTokenFromCredentials` to read the APIKey, in scripts.  |
| Impersonate Connector Account  |Unchecked| When true, the synchronization service runs the Windows PowerShell scripts in the context of the credentials supplied. When possible, it is recommended that the **$Credentials** parameter is passed to each script is used instead of impersonation.|
| Load User Profile When Impersonating |Unchecked|Instructs Windows to load the user profile of the connector’s credentials during impersonation. If the impersonated user has a roaming profile, the connector does not load the roaming profile.|
| Logon Type When Impersonating  |None|Logon type during impersonation. For more information, see the [dwLogonType](/windows/win32/api/winbase/nf-winbase-logonusera#parameters) documentation. |
|Signed Scripts Only |Unchecked|  If true, the Windows PowerShell connector validates that each script has a valid digital signature. If false, ensure that the Synchronization Service server’s Windows PowerShell execution policy is RemoteSigned or Unrestricted.|
|Common Module Script Name (with extension)|Common Module.psm1|The connector allows you to store a shared Windows PowerShell module in the configuration. When the connector runs a script, the Windows PowerShell module is extracted to the file system so that it can be imported by each script.|
|Common Module Script|[AD Sync PowerShell Connector Module code](https://github.com/microsoft/MIMPowerShellConnectors/blob/master/src/ECMA2HostCSV/Scripts/CommonModule.psm1) as value.  This module will be automatically created by the ECMA2Host when the connector is running. | I have extended this module by adding one function `Get-CSEntryChangeValue` for retrieving connector space object value.  |
|Validation Script|\<Blank\>|The Validation Script is an optional Windows PowerShell script that can be used to ensure that connector configuration parameters supplied by the administrator are valid.|
|Schema Script|[GetSchema code](https://github.com/microsoft/MIMPowerShellConnectors/blob/master/src/ECMA2HostCSV/Scripts/Schema%20Script.ps1) as value.| I have extended the out-of-the-box script to generate a log file, by adding one function `Write-DebugLog`. To make sure all the new attributes I have added in the `Schema.xml` file are imported for Person object type.|
|Additional Config Parameter Names|\<Blank\>|In addition to the standard configuration settings, you can define additional custom configuration settings that are specific to the instance of the Connector. These parameters can be specified at the connector, partition, or run step levels and accessed from the relevant Windows PowerShell script. We have not used it in this example.  |
|Additional Encrypted Config Parameter Names|\<Blank\> ||

<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorConnectivityPage.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

Once all the properties are set, click on **Next**.

### Capabilities Page

On the Capabilities Page, I have used the following attributes:

|Parameter|Value|Purpose|
|----|-----|-----|
|Distinguished Name Style|None|Indicates if the connector supports distinguished names and if so, what style. |
|Export Type|ObjectReplace|Determines the type of objects that are presented to the Export script.|
|Data Normalization|None|Instructs the Synchronization Service to normalize anchor attributes before they are provided to scripts. |
|Object Confirmation|NoAddAndDeleteConfirmation|This is ignored.|
|Use DN as Anchor|Unchecked|If the Distinguished Name Style is set to LDAP, the anchor attribute for the connector space is also the distinguished name. |
|Concurrent Operations of Several Connectors|UnChecked|When checked, multiple Windows PowerShell connectors can run simultaneously. |
|Partitions|Unchecked|When checked, the connector supports multiple partitions and partition discovery. |
|Hierarchy|Unchecked|When checked, the connector supports an LDAP style hierarchical structure. |
|Enable Import|Checked|When checked, the connector imports data via import scripts. |
|Enable Delta Import|Unchecked|When checked, the connector can request deltas from the import scripts. |
|Enable Export|Checked|When checked, the connector exports data via export scripts. |
|Enable Full Export|Unchecked|Not supported. This will be ignored.|
|No Reference Values In First Export Pass|Unchecked|When checked, reference attributes are exported in a second export pass. |
|Enable Object Rename|Unchecked|When checked, distinguished names can be modified. |
|Delete-Add As Replace|Unchecked|Not supported. This will be ignored.|
|Enable Export Password in First Pass|Unchecked|Not supported. This will be ignored.|

<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorCapabilitiesPage.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

Once all the properties are set, click on **Next**.

### Global Page

This is probably the most important configuration page. You will need to define your import and export scripts here.

On the Global Page, I have used the following attributes:

|Parameter|Value|
|-----|-----|
|Partition Script|\<Blank>|
|Hierarchy Script|\<Blank>|
|Begin Import Script|\<Blank>|
|Import Script|[Paste the Import Script Okta as the value](https://github.com/microsoft/MIMPowerShellConnectors/blob/master/src/ECMA2HostCSV/Scripts/Import%20Scripts.ps1)|
|End Import Script|\<Blank>|
|Begin Export Script|\<Blank>|
|Export Script|[Paste the Export Script Okta as the value](https://github.com/microsoft/MIMPowerShellConnectors/blob/master/src/ECMA2HostCSV/Scripts/Export%20Script.ps1)|
|End Export Script|\<Blank>|
|Begin Password Script|\<Blank>|
|Password Extension Script|\<Blank>|
|End Password Script|\<Blank>|

Let us take this opportunity to understand the import and export scripts.

The Goal of the import script is to get the data from the target application and transform it into CSEntryChange objects that can be imported into the Metaverse. The imported users are then processed and added to the import results.

Similarly, the goal of the export script is to export data from the ECMA2Host Cache to the target application. The exported users are then processed and added to the export results.


<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorGlobalPage.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

Once all the properties are set, click on **Next**.

### Partitions, Run Profiles, Export, FullImport

Keep the defaults and click **next**.

### Object types

Configure the object types tab with the information provided in the table.

|Parameter|Value|
|-----|-----|
|Target Object|Person|
|Anchor|login|
|DN|-dn-|

### Select Attributes

Add the following attributes to the connector. These values will be populated from the Schema.xml file.

- login
- firstName
- lastName
- nickName
- displayName
- email
- secondEmail
- profileUrl
- preferredLanguage
- userType
- organization
- title
- division
- department
- costCenter
- employeeNumber
- mobilePhone
- primaryPhone
- streetAddress
- city
- state
- zipCode
- countryCode
- status

### Deprovisioning

On the Deprovisioning page, you can specify if you want Microsoft Entra ID to remove users from the directory when they go out of scope of the application. In our use case, we will only delete objects from the target system when they are hard deleted. Therefore, select **None** under Disable flow and **Delete** under Delete flow.

If the status of a user is changed to Inactive, they will be set as `DEPROVISIONED` in the Target Application.

<div style="display: flex; justify-content: center; max-width: 500px;">
  <video controls>
    <source src="/assets/img/PowershellConnectorAttributePage.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

## Setting Up Entra ID On-Premises Application

Instantiate the **On-Premises ECMA App** app template in the Entra ID portal.

![Entra ID Connector](/assets/img/PowershellConnectorCreateOnPremApp.png)

To configure provisioning, change the Provisioning Mode to Automatic on the created enterprise app. Then, select your agent installed previously and click on Assign Agent(s).

Normally you need to wait 10 minutes before proceeding to the next step, but for this blog post, I will proceed to the next step. 

And to do that we will restart the following services.

- **Microsoft ECMA2Host Service**
- **Microsoft Azure AD Connect Provisioning Agent**

Once all the services are restarted, test the connectivity.
Provide the `tenant url` in the following format.

`https://localhost:8585/ecma2host_{CONNECTORNAME}/scim`

We have replaced it with the name of our connector `PowershellRestAPI` in the above URL.

`https://localhost:8585/ecma2host_PowershellRestAPI/scim`.For the token use the token you have used in the connector properties page.

![Entra ID Test Connector](/assets/img/PowershellConnectorGenerateTestCred.png)

At this time, ECMA2 Host Service will initiate a Full Import to retrieve all the users from the Target Application.

### Attribute Mapping and Implementation

All the schema attributes we have added are already available in the Entra ID portal, for selection as `Target Attribute`.

You can verify that by navigating to the **Attribute Mapping** tab selecting the [X]**Show advanced options**
checkbox and selecting the **Edit attribute list for the ScimOnPremises** link.

![Entra Attribute List](/assets/img/PowershellConnectorEntraAttributeList.png)

For our specific use case we have added the following mapping:

|Mapping type|Source attribute|Target attribute|
|-----|-----|-----|
| Direct |userPrincipalName |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:login|
| Direct |displayName |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:displayName|
| Direct |department |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:division|
| Direct |mail |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:email|
| Direct |givenName |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:firstName|
| Direct |surname |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:lastName|
| Direct |mobile |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:mobilePhone|
| Direct |preferredLanguage |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:preferredLanguage|
| Direct |jobTitle |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:title|
| Direct |employeeType |urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:userType|
|Direct|employeeId|urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:employeeNumber|
|Expression|Switch([IsSoftDeleted], , "False", "Active", "True", "Inactive")|urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:status|

## Testing the Connector

Let's start by assigning one user to the Application and Initiate On-Demand Provisioning.
From the Provisioning Log, we can verify user is successfully provisioned.

### Testing Provisioning Add Operation

- Step 1: Import User from Entra ID

![Step 1](/assets/img/PowershellConnectorProvisionAdd1.png)

- Step 2: Provisioning Service Determines if the User object is in scope for Provisioning. 
![PowerShellConnectorProvisioningAdd2](/assets/img/PowershellConnectorProvisionAdd2.png)

- Step 3: Provisioning Service Detemines the User is not present in our Target Application.
![PowerShellConnectorProvisioningAdd3](/assets/img/PowershellConnectorProvisionAdd3.png)
- Step 4: Provisioning Service Initiates the Provisioning Add Operation. Under the hood, the Export Script is called to create the user in the Target Application.
![PowerShellConnectorProvisioningAdd4](/assets/img/PowershellConnectorProvisionAdd4.png)

We can log in our Target Application and verify the user is created.

![PowerShellConnectorProvisioningAdd6](/assets/img/PowershellConnectorProvisionAdd5.png)

### Testing Provisioning Update Operation

Next, we will update the user in Entra ID and verify the user is updated in the Target Application. I habe disabled the user in Entra ID and initiated the On-Demand Provisioning.

![PowerShellConnectorProvisioningUpdate1](/assets/img/PowershellConnectorProvisionUpdate1.png)

In our `Export Script`, changing the status attribute will trigger `activate` or `deactivate` lifecycle operation in the Target Application for the userId.

We can verify in the target application that the user is now deactivated.

![PowerShellConnectorProvisioningUpdate2](/assets/img/PowershellConnectorProvisionUpdate2.png)

After 30 days Entra ID will hard delete the user from the Target Application. Which will result in `Delete-OktaUser` function call and result in `delete` operation in the Target Application.

## Conclusion

In this blog post, we have learned how to set up the Entra ID PowerShell Connector. We have also learned how to modify the Schema.xml file to match user attributes in your target application. We have also learned how to modify the Import.ps1 file to Import user data from your target application to Entra ID Provisioning Service. We have also learned how to modify the Export.ps1 file to Export user data from Entra ID Provisioning Service to your target application. We have also learned how to modify the Common Module Script to add Custom Functions.

Powershell is a great tool when it comes to integrating with applications and has strong community support. Integrating it with **Entra ID Provisioning Service** open up a lot of possibilities for IT professionals to integrate with legacy applications and cloud-based Entra ID IAM solution.


## Scripts

### Import Script

```powershell
# Import Script
<#
.SYNOPSIS
    This script imports provisioned and deprovisioned users from Okta into a Metaverse.

.DESCRIPTION
    The script retrieves user data from Okta using the Okta API and transforms it into CSEntryChange objects
    that can be imported into the Metaverse. The imported users are then processed and added to the import results.

.PARAMETER ConfigParameters
    The configuration parameters for the script.

.PARAMETER Schema
    The schema of the Metaverse.

.PARAMETER OpenImportConnectionRunStep
    The run step for opening the import connection.

.PARAMETER GetImportEntriesRunStep
    The run step for retrieving the import entries.

.PARAMETER PSCredential
    The credentials for authenticating with the Okta API.

.FUNCTIONS
    The script contains the following functions:
    - Write-DebugLog: Writes a debug log message to a log file.
    - GetTokenFromCredentials: Retrieves the token from the provided credentials.
    - Import-OktaProvisionedUsers: Imports provisioned and deprovisioned users from Okta.

.OUTPUTS
    The script outputs the import results, which include the CSEntryChange objects to be imported into the Metaverse.

.EXAMPLE
    Import-OktaProvisionedUsers -ConfigParameters $ConfigParameters -Schema $Schema -OpenImportConnectionRunStep $OpenImportConnectionRunStep -GetImportEntriesRunStep $GetImportEntriesRunStep -PSCredential $PSCredential
#>

param(
    [System.Collections.ObjectModel.KeyedCollection[string, Microsoft.MetadirectoryServices.ConfigParameter]]
    [ValidateNotNull()]
    $ConfigParameters,
    [Microsoft.MetadirectoryServices.Schema]
    [ValidateNotNull()]
    $Schema,
    [Microsoft.MetadirectoryServices.OpenImportConnectionRunStep]
    $OpenImportConnectionRunStep,
    [Microsoft.MetadirectoryServices.ImportRunStep]
    $GetImportEntriesRunStep,
    [pscredential]
    $PSCredential
)

#region Functions

function Write-DebugLog {
    param
    (
        [string]$Message
    )
  
    $logPath = "C:\ImportScript.log"
    $logMessage = "$(Get-Date -Format "yyyy-MM-dd HH:mm:ss") - $Message"
    Add-Content -Path $logPath -Value $logMessage
  
}

function GetTokenFromCredentials {
    param (
      [System.Management.Automation.PSCredential]$Credentials
    )
     Write-DebugLog "Retrieving TOKEN"
    $token = $Credentials.GetNetworkCredential().Password
    Write-DebugLog "TOKEN Retrieval completed"
    return $token
  }

#endregion

#region SCIM Functions

function Import-OktaProvisionedUsers {
    param (
        
    )
    $Token = GetTokenFromCredentials -Credentials $PSCredential

    $headers = @{
        "Accept" = "application/json"
        "Content-Type" = "application/json"
        "Authorization" = "SSWS $Token"
        "Cookie" = "JSESSIONID=74D0E3DCA57D73E89D7019244B2255D1"
    }
    $results = New-Object System.Collections.Generic.List[PSObject]

    $response = Invoke-RestMethod 'https://trial-8839557-admin.okta.com//api/v1/users?filter=status eq "PROVISIONED"' -Method 'GET' -Headers $headers
    foreach ($user in $response) {
        $results.Add([PSCustomObject]@{
            "status" = if ($user.status -eq "PROVISIONED") { "Active" } else { "Inactive" }
            "firstName" = $user.Profile.firstName
            "lastName" = $user.Profile.lastName
            "nickName" = $user.Profile.nickName
            "displayName" = $user.Profile.displayName
            "email" = $user.Profile.email
            "secondEmail" = $user.Profile.secondEmail
            "profileUrl" = $user.Profile.profileUrl
            "preferredLanguage" = $user.Profile.preferredLanguage
            "userType" = $user.Profile.userType
            "organization" = $user.Profile.organization
            "title" = $user.Profile.title
            "division" = $user.Profile.division
            "department" = $user.Profile.department
            "costCenter" = $user.Profile.costCenter
            "employeeNumber" = $user.Profile.employeeNumber
            "mobilePhone" = $user.Profile.mobilePhone
            "primaryPhone" = $user.Profile.primaryPhone
            "streetAddress" = $user.Profile.streetAddress
            "city" = $user.Profile.city
            "state" = $user.Profile.state
            "zipCode" = $user.Profile.zipCode
            "countryCode" = $user.Profile.countryCode
            "login" = $user.Profile.login
            
        })
    }
    $responseDeprovisioned = Invoke-RestMethod 'https://trial-8839557-admin.okta.com//api/v1/users?filter=status eq "DEPROVISIONED"' -Method 'GET' -Headers $headers
    foreach ($user in $responseDeprovisioned) {
        $results.Add([PSCustomObject]@{
            "status" = if ($user.status -eq "DEPROVISIONED") { "Inactive" } else { "Active" }
            "firstName" = $user.Profile.firstName
            "lastName" = $user.Profile.lastName
            "nickName" = $user.Profile.nickName
            "displayName" = $user.Profile.displayName
            "email" = $user.Profile.email
            "secondEmail" = $user.Profile.secondEmail
            "profileUrl" = $user.Profile.profileUrl
            "preferredLanguage" = $user.Profile.preferredLanguage
            "userType" = $user.Profile.userType
            "organization" = $user.Profile.organization
            "title" = $user.Profile.title
            "division" = $user.Profile.division
            "department" = $user.Profile.department
            "costCenter" = $user.Profile.costCenter
            "employeeNumber" = $user.Profile.employeeNumber
            "mobilePhone" = $user.Profile.mobilePhone
            "primaryPhone" = $user.Profile.primaryPhone
            "streetAddress" = $user.Profile.streetAddress
            "city" = $user.Profile.city
            "state" = $user.Profile.state
            "zipCode" = $user.Profile.zipCode
            "countryCode" = $user.Profile.countryCode
            
        })
    }
    
    Write-DebugLog "Imported Users from Okta"
    $responseJSON = $results | ConvertTo-Json
    Write-DebugLog "$responseJSON"
    $results

    
}


#endregion

Set-PSDebug -Strict
Write-DebugLog "Starting Import Script"

$commonModule = (Join-Path -Path ([Microsoft.MetadirectoryServices.MAUtils]::MAFolder) -ChildPath $ConfigParameters['Common Module Script Name (with extension)'].Value)
Import-Module -Name $commonModule -Verbose:$false -ErrorAction Stop

$importResults = New-Object -TypeName 'Microsoft.MetadirectoryServices.GetImportEntriesResults'

$csEntries = New-Object -TypeName 'System.Collections.Generic.List[Microsoft.MetadirectoryServices.CSEntryChange]'

$columnsToImport = $Schema.Types[0].Attributes

Write-DebugLog "Loaded $($columnsToImport.Count) attributes to import"
foreach ($column in $columnsToImport)
{
    Write-DebugLog "Attribute: $($column.Name)"
}

$recordsToImport = Import-OktaProvisionedUsers

Write-DebugLog "Imported $($recordsToImport.Count) records"


foreach ($record in $recordsToImport)
{

    Write-DebugLog 'Starting new record'

    ##TODO: Handle a missing anchor (what exception to throw?)

    $foundValidColumns = $false

    $entrySchema = $Schema.Types[0];
    $csEntry = New-xADSyncPSConnectorCSEntryChange -ObjectType $entrySchema.Name -ModificationType Add

    foreach ($column in $columnsToImport)
    {

        $columnName = $column.Name

        Write-DebugLog "Processing column $columnName"

        if ($record.$columnName)
        {

            Write-DebugLog 'Found column'

            $foundValidColumns = $true

            ##TODO: Support multivalue?

            $anchorAttrName = $entrySchema.AnchorAttributes[0].Name
            $value = [string]$record.$columnName

            Write-DebugLog "$columnName with value equal $value"


            if ($columnName -eq $anchorAttrName) {


                $csEntry.AnchorAttributes.Add([Microsoft.MetadirectoryServices.AnchorAttribute]::Create($columnName, $value))
            }


            $csEntry | Add-xADSyncPSConnectorCSAttribute -ModificationType Add -Name $columnName -Value ([Collections.IList]($record.$columnName.Split(";")))

        }

    }

    if ($foundValidColumns)
    {

        Write-DebugLog 'Publishing CSEntryChange'

        $csEntries.Add($csEntry)

    }

    Write-DebugLog 'Record completed'

}

##TODO: Support paging

$importResults.CSEntries = $csEntries

$importResults.MoreToImport = $false

Write-Output $importResults


```

### Export Script

```powershell
# Export Script
<#
.SYNOPSIS
This script exports data from a Metadirectory Services (MIM) connector to Okta.

.DESCRIPTION
The script exports data from a Metadirectory Services (MIM) connector to Okta by making REST API calls to the Okta API. It retrieves the necessary credentials, creates custom PS objects, and performs operations such as adding, updating, and deleting Okta users.

.PARAMETER ConfigParameters
A collection of configuration parameters required for the script.

.PARAMETER Schema
The schema of the Metadirectory Services (MIM) connector.

.PARAMETER OpenExportConnectionRunStep
The run step for opening the export connection.

.PARAMETER CSEntries
A list of CSEntryChange objects representing the changes to be exported.

.PARAMETER PSCredential
The PowerShell credential object containing the necessary credentials for authentication.

.EXAMPLE
.\Export Script Okta.ps1 -ConfigParameters $ConfigParameters -Schema $Schema -OpenExportConnectionRunStep $OpenExportConnectionRunStep -CSEntries $CSEntries -PSCredential $PSCredential
#>

param(
  [System.Collections.ObjectModel.KeyedCollection[string, Microsoft.MetadirectoryServices.ConfigParameter]]
  $ConfigParameters,
  [Microsoft.MetadirectoryServices.Schema]
  $Schema,
  [Microsoft.MetadirectoryServices.OpenExportConnectionRunStep]
  $OpenExportConnectionRunStep,
  [System.Collections.Generic.IList[Microsoft.MetaDirectoryServices.CSEntryChange]]
  $CSEntries,
  [pscredential]
  $PSCredential
)

Set-PSDebug -Strict

# Rest of the code...
param(
  [System.Collections.ObjectModel.KeyedCollection[string, Microsoft.MetadirectoryServices.ConfigParameter]]
  $ConfigParameters,
  [Microsoft.MetadirectoryServices.Schema]
  $Schema,
  [Microsoft.MetadirectoryServices.OpenExportConnectionRunStep]
  $OpenExportConnectionRunStep,
  [System.Collections.Generic.IList[Microsoft.MetaDirectoryServices.CSEntryChange]]
  $CSEntries,
  [pscredential]
  $PSCredential
)

Set-PSDebug -Strict


$commonModule = (Join-Path -Path ([Microsoft.MetadirectoryServices.MAUtils]::MAFolder) -ChildPath $ConfigParameters['Common Module Script Name (with extension)'].Value)
Import-Module -Name $commonModule -Verbose:$false -ErrorAction Stop

#region function

function CreateCustomPSObject {

  param
  (
    $PropertyNames = @()
  )

  $template = New-Object -TypeName System.Object

  foreach ($property in $PropertyNames) {

    $template | Add-Member -MemberType NoteProperty -Name $property -Value $null

  }

  return $template
}


function Write-DebugLog {
  param
  (
    [string]$Message
  )

  $logPath = "C:\ExportScript.log"
  $logMessage = "$(Get-Date -Format "yyyy-MM-dd HH:mm:ss") - $Message"
  Add-Content -Path $logPath -Value $logMessage

}

function GetTokenFromCredentials {
  param (
    [System.Management.Automation.PSCredential]$Credentials
  )
  Write-DebugLog "Retrieving TOKEN"
  $token = $Credentials.GetNetworkCredential().Password
  Write-DebugLog "TOKEN Retrieval completed"
  return $token
}



#region OKTA Functions
function Add-OktaProvisionedUser {
  param (
    $baseObject
  )

  $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
  $Token = GetTokenFromCredentials -Credentials $PSCredential
  $headers.Add("Accept", "application/json")
  $headers.Add("Content-Type", "application/json")
  $headers.Add("Authorization", "SSWS $Token")
  $headers.Add("Cookie", "JSESSIONID=C0B3B05DB3903768410F4B0AD4EC7BE3")
  
  $body = @"
  {
          `"profile`": {
                  `"login`": `"Vasco.brock@example.com`",
                  `"firstName`": `"Vasco`",
                  `"lastName`": `"Brock`",
                  `"nickName`": `"Vasco`",
                  `"displayName`": `"Vasco Brock`",
                  `"email`": `"Vasco.brock@example.com`",
                  `"secondEmail`": `"Vasco@example.org`",
                  `"profileUrl`": `"http://www.example.com/profile`",
                  `"userType`": `"Employee`",
                  `"organization`": `"Okta`",
                  `"title`": `"Director`",
                  `"division`": `"R&D`",
                  `"department`": `"Engineering`",
                  `"costCenter`": `"10`",
                  `"employeeNumber`": `"187`",
                  `"mobilePhone`": `"+1-555-415-1337`",
                  `"primaryPhone`": `"+1-555-514-1337`",
                  `"streetAddress`": `"301 Brannan St.`",
                  `"city`": `"San Francisco`",
                  `"state`": `"CA`",
                  `"zipCode`": `"94107`",
                  `"countryCode`": `"US`"
          }
  }
"@
  $body = $body.Replace("Vasco.brock@example.com", $($baseObject.login))
  $body = $body.Replace("Vasco", $($baseObject.firstName))
  $body = $body.Replace("Brock", $($baseObject.lastName))
  $body = $body.Replace("Vasco", $($baseObject.nickName))
  $body = $body.Replace("Vasco Brock", $($baseObject.displayName))
  $body = $body.Replace("Vasco.brock@example.com", $($baseObject.email))
  $body = $body.Replace("Vasco@example.org", $($baseObject.secondEmail))
  $body = $body.Replace("http://www.example.com/profile", $($baseObject.profileUrl))
  #$body = $body.Replace("en-US", $($baseObject.preferredLanguage))
  $body = $body.Replace("Employee", $($baseObject.userType))
  $body = $body.Replace("Okta", $($baseObject.organization))
  $body = $body.Replace("Director", $($baseObject.title))
  $body = $body.Replace("R&D", $($baseObject.division))
  $body = $body.Replace("Engineering", $($baseObject.department))
  $body = $body.Replace("10", $($baseObject.costCenter))
  $body = $body.Replace("187", $($baseObject.employeeNumber))
  $body = $body.Replace("+1-555-415-1337", $($baseObject.mobilePhone))
  $body = $body.Replace("+1-555-514-1337", $($baseObject.primaryPhone))
  $body = $body.Replace("301 Brannan St.", $($baseObject.streetAddress))
  $body = $body.Replace("San Francisco", $($baseObject.city))
  $body = $body.Replace("CA", $($baseObject.state))
  $body = $body.Replace("94107", $($baseObject.zipCode))
  $body = $body.Replace("US", $($baseObject.countryCode))
  Write-DebugLog -Message "Body for AddObject:\n $body"
  
  try {
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    $response = Invoke-RestMethod 'https://trial-8839557-admin.okta.com//api/v1/users?activate=true' -Method 'POST' -Headers $headers -Body $body
    $response = $response | ConvertTo-Json 
    Write-DebugLog -Message "Response: $response"
  
          
  }
  catch {
    Write-DebugLog -Message "Error: $_"
  }

  
}

function Update-OktaUser {
  param (
      
    [Parameter(Mandatory = $true)]
    [string]$attributeName,

    [Parameter(Mandatory = $true)]
    [string]$attributeValue,

    [Parameter(Mandatory = $true)]
    [string]$userId
  )

  $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
  $token = GetTokenFromCredentials -Credentials $PSCredential
  $headers.Add("Accept", "application/json")
  $headers.Add("Content-Type", "application/json")
  $headers.Add("Authorization", "SSWS $token")
  $headers.Add("Cookie", "JSESSIONID=D6E791D528C893BC8B32EDDA0FD2A8F7")

  if ($attributeName -eq "status") {
    if ($attributeValue -eq "Active") {
      try {
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        $response = Invoke-RestMethod "https://trial-8839557-admin.okta.com//api/v1/users/$userId/lifecycle/activate?sendEmail=false" -Method 'POST' -Headers $headers
        $response = $response | ConvertTo-Json 
        Write-DebugLog -Message "Response: $response"
        
      }
      catch {
        Write-DebugLog -Message "Error: $_"
      }
      
    }
    if ($attributeValue -eq "Inactive") {
      try {
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        $response = Invoke-RestMethod "https://trial-8839557-admin.okta.com//api/v1/users/$userId/lifecycle//deactivate?sendEmail=false" -Method 'POST' -Headers $headers
        $response = $response | ConvertTo-Json 
        Write-DebugLog -Message "Response: $response"
        
      }
      catch {
        Write-DebugLog -Message "Error: $_"
      }
    }
  }
  else {
    $body = @{
      profile = @{
        $attributeName = $attributeValue
      }
    } | ConvertTo-Json
    try {
      [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
      $response = Invoke-RestMethod "https://trial-8839557-admin.okta.com//api/v1/users/$userId" -Method 'POST' -Headers $headers -Body $body
      $response = $response | ConvertTo-Json 
      Write-DebugLog -Message "Response: $response"
  
        
    }
    catch {
      Write-DebugLog -Message "Error: $_"
    }

  }
  

  
}

function Delete-OktaUser {
  param (
    
    [Parameter(Mandatory = $true)]
    [string]$userId

  )

  $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
  $token = GetTokenFromCredentials -Credentials $PSCredential
  $headers.Add("Accept", "application/json")
  $headers.Add("Content-Type", "application/json")
  $headers.Add("Authorization", "SSWS $token")
  $headers.Add("Cookie", "JSESSIONID=D6E791D528C893BC8B32EDDA0FD2A8F7")


  
  try {
    
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    $url = "https://trial-8839557-admin.okta.com//api/v1/users/$userId"
    Write-DebugLog -Message "URL: $url"
    $response = Invoke-RestMethod -Uri $url -Method 'DELETE' -Headers $headers
    $response = $response | ConvertTo-Json 
    Write-DebugLog -Message "Response: $response"

   
  }
  catch {
    Write-DebugLog -Message "Error: $_"
  }


}

#endregion

Write-DebugLog -Message "-----------------------------------------------------------------------------"
Write-DebugLog -Message "Starting Export Script Main Function"

$csentryChangeResults = New-Object "System.Collections.Generic.List[Microsoft.MetadirectoryServices.CSEntryChangeResult]"


$columnsToExport = @()

foreach ($attribute in $Schema.Types[0].Attributes) {

  $columnsToExport += $attribute.Name

  Write-DebugLog -Message "Added attribute $($attribute.Name) to export list"

}



Write-DebugLog -Message "Processing object $($entry.Identifier)"


foreach ($entry in $CSEntries) {
  
  

  Write-DebugLog -Message "-------------------------------------------------------------------------------------------------------"
    
  foreach ($attributeName in $entry.ChangedAttributeNames) {
    Write-DebugLog -Message "Attribute: $attributeName"
    Write-DebugLog -Message "Value: $($entry.AttributeChanges[$attributeName].ValueChanges[0].Value)"
    $anchorAttributeValue = $entry.AnchorAttributes[0].Value.ToString();
    Write-DebugLog -Message "Anchor Attribute: $($entry.AnchorAttributes[0].Name)"
    Write-DebugLog -Message "Anchor Value: $anchorAttributeValue"
    if ($entry.ObjectModificationType -eq 'Replace') {
      Update-OktaUser -attributeName $attributeName -attributeValue $entry.AttributeChanges[$attributeName].ValueChanges[0].Value -userId $anchorAttributeValue


    }
    
    
  }
 
  Write-DebugLog -Message "Processing object $($entry.Identifier). ObjectModificationType $($entry.ObjectModificationType)"

  [bool]$objectHasAttributes = $false

  
  $baseObject = CreateCustomPSObject -PropertyNames $columnsToExport

  if ($entry.ObjectModificationType -eq 'Replace') {
    $anchorAttributeName = $entry.AnchorAttributes[0].Name;
    $anchorAttributeValue = $entry.AnchorAttributes[0].Value.ToString();
    
    

    foreach ($attribute in $entry.ChangedAttributeNames) {
      
      $value = Get-CSEntryChangeValue -CSEntryChange $entry -AttributeName $attribute
      Write-DebugLog -Message " Changed Attribute Value is : $value"
      
      

    }
  }


  if ($entry.ObjectModificationType -ne 'Delete') {

    foreach ($attribute in $columnsToExport) {

      if (($entry.AttributeChanges.Contains($attribute)) -eq $false -and ($entry.AnchorAttributes.Contains($attribute) -eq $false)) {

        continue

      }


      if ($entry.AnchorAttributes[$attribute].Value) {

        $baseObject.$attribute = $entry.AnchorAttributes[$attribute].Value

        $objectHasAttributes = $true

      }

      elseif ($entry.AttributeChanges[$attribute].ValueChanges[0].Value) {

        $baseObject.$attribute = ($entry.AttributeChanges[$attribute].ValueChanges | Select-Object -Expand Value) -join ";"

        $objectHasAttributes = $true

      }
      elseif ($entry.AttributeChanges[$attribute].DataType -eq "Boolean") {
        $baseObject.$attribute = ($entry.AttributeChanges[$attribute].ValueChanges | Select-Object -Expand Value) -join ";"

        

      }

    }

    if ($objectHasAttributes) {

      foreach ($property in $baseObject.PSObject.Properties) { 
        if ($property.Value -eq $null) {
          $baseObject.($property.Name) = ""
        }
      }
      
      
      foreach ($property in $baseObject.PSObject.Properties) {
        Write-DebugLog -Message "Added $($property.Name) with value $($property.Value)"
      }
      if ($entry.ObjectModificationType -eq 'Add') {
        Write-DebugLog -Message "Adding user to OKTA with Email $($baseObject.Email) and DisplayName $($baseObject.DisplayName) and UserName $($baseObject.UserName) and AzureObjectID $($baseObject.AzureObjectID)"
        Add-OktaProvisionedUser -baseObject $baseObject
      }

      

    }

  }
  else {
    $anchorAttributeName = $entry.AnchorAttributes[0].Name;
    $anchorAttributeValue = $entry.AnchorAttributes[0].Value.ToString();
    Write-DebugLog -Message "Delete the object with attribute '$($anchorAttributeName)' equals '$($anchorAttributeValue)'"
    Delete-OktaUser -userId $anchorAttributeValue
  }

  $csentryChangeResult = [Microsoft.MetadirectoryServices.CSEntryChangeResult]::Create($entry.Identifier, $null, "Success")
  $csentryChangeResults.Add($csentryChangeResult)

  Write-DebugLog -Message "Completed processing object $($entry.Identifier)"

}

$closedType = [type]"Microsoft.MetadirectoryServices.PutExportEntriesResults"

return [Activator]::CreateInstance($closedType, $csentryChangeResults)
```

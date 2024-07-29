---
layout: post
title: SCIMming into the Future - Provisioning On-Premises Apps with Azure AD Identity Governance - Part 2
subtitle: Automate provisioning and governance of your on-premises applications
cover-img: /assets/img/SCIM-Part2-4.png
thumbnail-img:  /assets/img/SCIM-Part2-4.png
share-img: /assets/img/SCIM-Part2-4.png
tags: [AzureAD,IdentityGovernance,SCIMProvisioning,OnPremisesApplications,MicrosoftEntra,UserManagement,HybridEnvironment,SCIM]
---
# SCIMming into the Future: Provisioning On-Premises Apps with Azure AD Identity Governance - Part 2

Continuing from the first [part](https://www.sitavog.tk/2023-02-12-SCIMming-into-the-Future-Provisioning-On-Premises-Apps-with-Azure-AD-Identity-Governance-Part-1/), in this is the part of the blog where we will install the [test SCIM application](https://aka.ms/scimreferencecode).

Follow this [guide](https://github.com/AzureAD/SCIMReferenceCode/wiki/Deploy-on-Azure-App-Service) to publish the sample web app from Microsoft in Azure App Services.

Once you publish the app, you will need to make changes to the appsettings. Navigate to the application in **Azure App Services** > **Configuration**, click on **New application setting** to add the ***Token__TokenIssuer*** setting with the value "https://sts.windows.net/***tenant_id***/". (replace ***tenant_id*** by your Azure AD tenant id). 

We are egoing to test the application with Azure AD, ***ASPNETCORE_ENVIRONMENT*** value to **'Development'**. 

In the app setting configuration file , you need to add the following settings.

```json
[
  {
    "name": "ASPNETCORE_ENVIRONMENT",
    "value": "Development",
    "slotSetting": false
  },
  {
    "name": "Token__TokenIssuer",
    "value": "https://sts.windows.net/a015f467-c5c4-****-****-************/",
    "slotSetting": false
  },
  {
    "name": "WEBSITE_NODE_DEFAULT_VERSION",
    "value": "6.9.1",
    "slotSetting": false
  }
]
```

## **Install AZURE AD Provisioning Agent**
Next, you will have to install the Azure AD Provisioning agent within your on-prem network in a server name SCIMVM. Your on-prem SCIM application , should be accessible from the provisioning Agent.

![Screenshot-of-Provisioning-Agent-EXCALIDRAW](/assets/img/SCIM-Part2-4.png)

1. [Download](https://aka.ms/OnPremProvisioningAgent) the provisioning agent and copy it onto the virtual machine or server that your SCIM application endpoint is hosted on.
![Screenshot-of-Provisioning-Agent-Download](/assets/img/SCIM-Part2-5.jpg)

 2. Run the provisioning agent installer, agree to the terms of service, and select **Install**.
 ![Screenshot-of-Provisioning-Agent-Install](/assets/img/SCIM-Part2-6.jpg)


 3. Once installed, locate  and launch the **AAD Connect Provisioning Agent wizard**, and when prompted for an extensions select **On-premises provisioning**.
 ![Screenshot-of-Provisioning-Agent-Install-Select-OnPrem-Provision](/assets/img/SCIM-Part2-7.jpg)
 4. For the agent to register itself with your tenant, provide credentials for an Azure AD admin with Hybrid administrator or global administrator permissions.
 5. Select **Confirm** to confirm the installation was successful.
 ![Screenshot-of-Provisioning-Agent-Install-Confirm](/assets/img/SCIM-Part2-8.jpg)


## **Provisioning to SCIM-enabled application**
Once the agent is installed, no further configuration is necesary on-prem, and all provisioning configurations are then managed from the portal. Repeat the below steps for every on-premises application being provisioned via SCIM.
 
 1. In the Azure portal navigate to the Enterprise applications and add the **On-premises SCIM app** from the gallery.
 ![Screenshot of Selecting App.](/assets/img/SCIM-Part2-13.png)


 2. From the left hand menu navigate to the **Provisioning** option and select **Get started**.
 3. Select **Automatic** from the dropdown list and expand the **On-Premises Connectivity** option.
 4.  Select the agent that you installed from the dropdown list and select **Assign Agent(s)**.
  ![Screenshot of Selecting Agent.](/assets/img/SCIM-Part2-10.jpg)

 5.  Now either wait 10 minutes or restart the **Microsoft Azure AD Connect Provisioning Agent** before proceeding to the next step & testing the connection.

 6.  In the **Tenant URL** field, provide the SCIM endpoint URL for your application. The URL is typically unique to each target application and must be resolveable by DNS.  In our test application the Tenant URL field, enter the URL of the application's SCIM endpoint. In the following format. ***https://{{Server}}/{{Api}}***
  Value for **Server** is the URL of the Web App hosted in Azure App Service.
 Value for **Api** is **scim**. 
 ![Screenshot that shows SCIM URL.](/assets/img/SCIM-Part2-2.jpg)



 7.  Select **Test Connection**, and save the credentials. The application SCIM endpoint must be actively listening for inbound provisioning requests, otherwise the test will fail. Use the steps [here](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/on-premises-ecma-troubleshoot#troubleshoot-test-connection-issues) if you run into connectivity issues.
 ![Screenshot that shows Test Connection is Successful.](/assets/img/SCIM-Part2-12.jpg)

 8.  Configure any attribute mappings or scoping rules required for your application. We are going to add the following attribute mapping to distinguish between Extrernal user type and Internal employees.
![Screenshot of User Attribute Mapping.](/assets/img/SCIM-Part2-3.jpg)

 9. Toggle the Provisioning Status button On and Save the Configuration.

In the last part we will configure the following.

1. We will Create Access Package and automatic assignment policy to assign the application to new hires (provisioning).

2. Create multi-staged access review to remove access for users who no longer require access to the application.




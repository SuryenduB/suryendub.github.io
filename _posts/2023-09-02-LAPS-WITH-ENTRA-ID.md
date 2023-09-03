---
layout: post
title: How to Implement and Manage LAPS for Microsoft Entra ID and Intune
subtitle :  A step-by-step guide to securing your local administrator passwords with LAPS
cover-img: /assets/img/LAPSTitle.jpg
thumbnail-img: /assets/img/LAPSTitle.jpg
share-img: /assets/img/LAPSTitle.jpg
tags: [ Azure Active Directory, EntraID, MicrosoftEntra , Security , LAPS]

---

# Title

## Implement and manage LAPS for Microsoft Entra ID

When LAPS (Local Administrator Password Solution) is not implemented, administrators often resort to two insecure practices for managing local administrator access on computers:

1. **Shared Local Admin Password:** In this scenario, administrators set a single, uniform local administrator password across multiple computers. This practice poses a significant security risk because it lacks granularity and accountability. The reasons for its insecurity are as follows:

   - **Uniformity:** Using a single shared password for all computers means that if it's compromised on one system, it's compromised on all, exposing the entire network to potential breaches.

   - **Limited Accountability:** When multiple administrators have access to the same password, it becomes challenging to trace who performed specific actions or changes on a given computer, hindering accountability.

   - **Difficulty in Rotation:** Regular password rotation, a crucial security practice, becomes cumbersome when managing shared passwords across numerous systems. Stale passwords increase vulnerability.

2. **Using Normal Credentials as Local Admin:** Some administrators may choose to log in as local administrators on remote systems using their standard domain credentials. While this approach may seem convenient, it introduces security risks, particularly regarding LSASS credential theft:

   - **Credential Exposure:** When administrators use their domain credentials on remote systems, these credentials are sent across the network and stored temporarily in LSASS memory. This exposure creates an opportunity for attackers to capture and exploit these credentials.

   - **LSASS Vulnerability:** The Local Security Authority Subsystem Service (LSASS) stores credentials in memory during the authentication process. Malicious actors can employ techniques like Pass-the-Hash (PtH) attacks to steal these stored credentials, compromising administrator accounts and potentially gaining unauthorized access.

In both cases, the absence of LAPS and secure local administrator management practices increases the organization's susceptibility to security breaches and unauthorized access. Implementing LAPS helps mitigate these risks by automating the management of unique, regularly rotated local administrator passwords, enhancing security and accountability across the network.

Password management for administrator accounts on AD DS or Azure AD–joined computers is a significant problem for Windows administrators. Implementing Local Administrator Password Solution (LAPS) is one solution.

 LAPS enables you to secure and help protect your Windows devices’ local admin passwords. Features include the ability to back up passwords and auto-rotate passwords. You must configure two related settings to enable and use LAPS in your Azure AD tenant:

 • Enable LAPS in Microsoft Entra
 • Configure LAPS using Intune

## Enabling LAPS in Microsoft Entra

 Before you can implement LAPS, you must enable it within your Azure AD tenant. You do this using the Microsoft Entra admin center. Use the following procedure:

 1. Open Microsoft Entra admin center.
 2. Expand Azure Active Directory in the navigation pane.
 3. Expand Devices and then select All devices.
 4. Click Device settings.
 5. Under the Local administrator settings, heading turn on the Enable Azure AD Local Administrator Password Solution (LAPS) setting.
![Figure Enable LAPS in Entra ID](/assets/img/LAPS1.jpg)


I consistently seek to achieve configurations through Microsoft Graph API calls, even when UI options are available, offers automation, consistency, version control, scalability, security, and enhanced auditing capabilities.

In this case however Microsoft Document for LAPS setting is inadequate.
Commandlet `Update-MgBetaPolicyDeviceRegistrationPolicy` is not working as expected.
I tried runnung with the Debug Switch to realize it is sending `PATCH` method however as per the [documentation](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/important-update-to-deviceregistrationpolicy-resource-type-for/ba-p/3912000) we will require `PUT` method and send all the properties.

```powershell
# Import the JSON file as a PowerShell object
$bodyParams = Get-Content $jsonFilePath -Raw 

$LocalAdminPwd = @{
    "IsEnabled" = "true"

}


Update-MgBetaPolicyDeviceRegistrationPolicy  -LocalAdminPassword $LocalAdminPwd -Debug

```

![HTTP PATCH Method in Update-MgBetaPolicyDeviceRegistrationPolicy](/assets/img/LAPS2.jpg)

To address the issue, I took the following steps: I obtained the list of settings for the Device Registration policy within Entra ID and then made the required modifications to activate the LAPS policy setting.

```powershell

#Get the Setting from URI and Store In a File
Get-MgBetaPolicyDeviceRegistrationPolicy | ConvertTo-JSON  | Out-File LAPSSettings.json


# Modify the JSON File and Update the Setting


$jsonFilePath = ".\LAPSSettings.json"
$json = Get-Content $jsonFilePath -Raw
$json = $json -replace '"IsEnabled": false', '"IsEnabled": true'



Connect-MgGraph -Scopes "Policy.ReadWrite.DeviceConfiguration"
Invoke-MgGraphRequest `
-Method PUT `
 -Uri "https://graph.microsoft.com/beta/policies/deviceRegistrationPolicy" `
 -Body $json `
 -ContentType "application/json" 
```

## Configuring LAPS using Intune

 Here are the revised instructions for creating an account protection policy in Intune:

**To create an account protection policy in Intune, follow these steps:**

1. Launch the Microsoft Intune admin center.
2. Navigate to Endpoint security in the sidebar and select Account protection.
3. In the details pane, click on the "Create Policy" button.
4. On the "Create a profile" page, choose "Windows 10 and later" from the Platform list.
5. From the Profile list, select "Local admin password solution (LAPS)."
6. Click the "Create" button.
7. In the "Create profile" wizard, provide a name on the Basics tab, and then click "Next."
8. Configure your desired settings on the Configuration settings tab.
9. Click "Next" and configure any scope tags if necessary.
10. Proceed to the next step to assign the policy to the appropriate device group.
11. Once you've made your assignments, click "Next" and then finalize the process by clicking "Create."

![Account Protection LAPS](https://learn.microsoft.com/en-us/mem/intune/protect/media/windows-laps-policy/create-laps-policy.png#lightbox)

![Specify Entra ID as Backup Directory](https://learn.microsoft.com/en-us/mem/intune/protect/media/windows-laps-policy/specify-the-backup-directory.png#lightbox)

## Read Device Password
If Everything goes right , you will be able to retrieve the password by following method.

To view the local administrator password in the Microsoft Entra admin center, follow these steps:

1. Navigate to **All devices** and select the appropriate device.
2. On the **Device | Properties** page, click the **Local administrator password recovery** tab.
3. Click the **Show local administrator password** link.

![Read LAPS Password](https://techcommunity.microsoft.com/t5/image/serverpage/image-id/463474i5B6E9E0092683EB5/image-dimensions/2000?v=v2&px=-1)

You can also review the password using the Microsoft Intune admin center:

1. In Intune, select **Devices** in the navigation pane.
2. Choose **All devices** and then select the appropriate device.
3. Click the **Local admin password** tab in the navigation pane.
4. Click the **Show local administrator password** link.

You can retrieve the password using the following Powershell Script.

```Powershell
Connect-MgGraph -Scopes "DeviceLocalCredential.Read.All"
Get-LapsAADPassword -DeviceIds "XXXXXX-8dcc-43bc-bef7-XXXXXXX" -IncludePasswords -AsPlainText

```

## Conclusion

In this article, we have discussed how to implement and manage LAPS for Microsoft Entra ID. LAPS is a powerful tool that can help you secure your local administrator passwords and improve the security of your organization. By following the steps outlined in this article, you can easily get started with LAPS and start reaping the benefits of improved security. Implementing and managing the Local Administrator Password Solution (LAPS) for Microsoft Entra ID is crucial for enhancing security and accountability in your organization's network. Without LAPS, administrators often resort to insecure practices like shared local admin passwords or using normal credentials as local admin, both of which pose significant security risks.

Shared local admin passwords lack granularity and accountability, making it challenging to trace actions and changes on specific devices. Additionally, rotating passwords becomes cumbersome, leading to security vulnerabilities due to stale passwords.

Using normal credentials as local admin introduces security risks, as credentials are exposed and stored in memory, potentially allowing attackers to capture and exploit them.

Here are some additional tips for implementing and managing LAPS:

- Make sure that all of your devices are enrolled in Azure AD.
- Create an account protection policy in Intune that specifies LAPS as the password management solution.
- Configure the LAPS policy settings to meet your organization's needs.
Regularly rotate the local administrator passwords.
- Test the LAPS solution to ensure that it is working properly.

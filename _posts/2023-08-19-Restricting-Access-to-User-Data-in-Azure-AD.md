---
layout: post
title: Restricting Access to User Data in Entra ID (Azure AD)

cover-img: /assets/img/taras-shypka-iFSvn82XfGo-unsplash.jpg
thumbnail-img: /assets/img/taras-shypka-iFSvn82XfGo-unsplash.jpg
share-img: /assets/img/taras-shypka-iFSvn82XfGo-unsplash.jpg
tags: [ Azure Active Directory, EntraID , AzureAD , MicrosoftGraph]

---

# 🔒 Restricting Access to User Data in Entra ID (Azure AD) 🔒



Let's address one of the key concerns: default permissions for member users. By default, Entra ID (Azure AD) allows any user to access and read data about other users, which can potentially expose sensitive information to unintended parties. 🕵️‍♂️

Imagine this scenario: if you want to take action on user objects based on their termination reason,or Perhaps you want to seamlessly transition an employee from a higher pay scale to a tax consultant application. 💼

**However, these details are confidential and should be restricted from other users.**

🚀 **One of the biggest drawbacks of the latest Lifestyle Workflow is that it does not allow custom security attributes for writing business logic.** 📊

In Entra ID, you can restrict the access to the default user settings to read all user attributes.

| Permission | Explaination |
| -------- | -------- |
| Read other users | This setting is available in Microsoft Graph and PowerShell only. Setting this flag to $false prevents all non-admins from reading user information from the directory. This flag doesn't prevent reading user information in other Microsoft services like Exchange Online.|

As explained in the previous table, this setting is only available through the Microsoft Graph API. I have written a small PowerShell snippet to restrict access for other users.

```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.Authorization"
$BodyParams = @{
    defaultUserRolePermissions = @{
        allowedToReadOtherUsers = $false

    }
}
Invoke-MgGraphRequest -Uri "https://graph.microsoft.com/beta/policies/authorizationPolicy/authorizationPolicy" -Method PATCH -Body $BodyParams 


```

You can verify the settings by running the Get Variant of the same command.


```powershell

$AuthorizationPolicy = Invoke-MgGraphRequest -Uri "https://graph.microsoft.com/beta/policies/authorizationPolicy/authorizationPolicy" -Method Get 
$AuthorizationPolicy['defaultUserRolePermissions'] | ConvertTo-Json

```

```json
{
  "allowedToCreateTenants": false,
  "allowedToCreateApps": false,
  "allowedToReadOtherUsers": false,
  "allowedToCreateSecurityGroups": false,
  "allowedToReadBitlockerKeysForOwnedDevice": true
}

```

## Note: This setting is meant for special circumstances, so Microsoft doesn't recommend setting the flag to $false

![Example image](/assets/img/taras-shypka-iFSvn82XfGo-unsplash.jpg)  

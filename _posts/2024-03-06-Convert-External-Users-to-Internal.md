---
layout: post
title: Simplifying the Bulk Conversion of External Users to Internal Users in Entra ID - Using PowerShell and Graph API
subtitle:  Simplifying the Conversion of External Users to Internal Users in Entra ID 
cover-img: /assets/img/ConvertToInternalThumb.png
thumbnail-img: /assets/img/ConvertToInternalThumb.png
share-img: /assets/img/ConvertToInternalTitle.jpeg
tags: [ IAM, IGA, EntraID, AzureAD, PowerShell, GraphAPI, B2B ]
readtime: true

---
# Simplifying the Conversion of External Users to Internal Users in Entra ID

## Table of Contents

- [Introduction](#introduction)
- [Background](#background)
- [Steps to Convert External Users to Internal Using Entra Admin Center](#steps-to-convert-external-users-to-internal-using-entra-admin-center)
  - [Converting an External user](#converting-an-external-user)
- [Convert using PowerShell and Graph API](#convert-using-powershell-and-graph-api)
- [Conclusion](#conclusion)

## Introduction

Previously, the process to convert internal employees was complicated and involved multiple steps to convert user types, UPN, and re-assign licenses. However, Microsoft has now introduced a new feature that allows for the conversion of external users to internal users with a single click.

## Background

For organizations going through acquisitions, mergers, and joint ventures, it is common to onboard employees from other organizations as external users. This helps ease them into the new environment without creating new accounts and minimizing disruption to their existing processes. However, over time, it becomes necessary to convert these newly acquired users into internal users.

With the new feature introduced by Microsoft, the process of converting external users to internal users has become simpler and more efficient.

## Steps to Convert External Users to Internal Using Entra Admin Center

You can follow the steps below to convert external users to internal users using the Azure AD portal.

### Converting an External user

You can convert external users to internal using the Microsoft Entra admin center. 

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com) as at least a **User Administrator**.

1. Browse to **Identity** > **Users** > **All users**.

1. Select an external user.
1. As shown in the image, select **Convert to internal user**

1. In the **Convert to internal user** section, you need to finalize a couple of steps:
    1. A **user principal name**, This value is the new UPN value for the user. 
    1. Choose whether you would like an auto generated password.
    1. Checkbox for **Change email address**, allows you to specify an optional new mail address for cloud users.
1. After reviewing the options and making and choices, you can choose **Convert**.

[![Convert External Users to Internal](https://img.youtube.com/vi/AT5JliCo0uA/0.jpg)](https://www.youtube.com/watch?v=AT5JliCo0uA)

### Convert using PowerShell and Graph API

However, if you want to convert a large number of users, you will need to utilize the Graph API. At the time of writing this post, there is no direct PowerShell commandlet to convert an external user to an internal user.

I have used the Graph API and `Invoke-MgGraphRequest` to convert external users to internal users. Below is the PowerShell script to convert an external user to an internal user.

- First create the variables for the user id, new UPN, and password for the user. Setting these as variables instead of hardcoding them
within the JSON gives us the option to run within a loop and change the
variables each time in the future.

```powershell
$userId = "3702a2bb-2d83-4f8d-ae18-b38d1e6998b6"
$newUPN = "suryendu.bhattacharyya@03z3s.onmicrosoft.com"
$password = "Mowo932415"
```

- Next we will create the JSON body for the request. This will include the new UPN and password for the user.

```powershell


$userJson = @"
{
    "userPrincipalName": "$newUPN",
    "passwordProfile": {
      "password": "$password",
      "forceChangePasswordNextSignIn": false
    },
    "mail": "$newUPN",
  }
"@
```

- Next we need to decide where to send the request. As this is a new feature, it is not yet available in the V1.0 endpoint. We will need to use the `beta` endpoint for this request. We will use the  `userid` variable to create the URL for the request. This will make the request dynamic and allow us to run the script for multiple users in a loop. `convertExternalToInternalMemberUser` is the operation we will run to convert the user.

```powershell
$aadurl = "https://graph.microsoft.com/beta/users/$userId/convertExternalToInternalMemberUser"

```

- Finally, we will send the request to the Graph API. We will use the `Invoke-MgGraphRequest` commandlet to send the `POST` request. This commandlet is part of the `Microsoft.Graph` module. This module is available from the PowerShell Gallery.

```powershell
##Convert External User to Internal User
write-output "Converting External User to Internal User"
$response = Invoke-MgGraphRequest -method POST -Uri $aadurl -Body $userJson -ContentType "application/json"
write-output "User Converted"
```
Now that we have the script, we can run it for each user we want to convert. Below is the result of the script.

**User Before Conversion** :
![User Before Conversion](/assets/img//ConvertToInternal.png)

**User After Conversion**
![User After Conversion](/assets/img//ConvertToInternal1.png)

## Conclusion

The new feature introduced by Microsoft has made the process of converting external users to internal users simpler and more efficient. This feature is especially useful for organizations that are going through acquisitions, mergers, and joint ventures. The ability to convert external users to internal users with a single click will help organizations streamline their processes and minimize disruption to their existing workflows. When testing external user conversion, Microsoft recommends that you use test accounts or accounts that wouldn't disrupt if they were to become unavailable.

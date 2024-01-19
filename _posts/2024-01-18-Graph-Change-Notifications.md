---
layout: post
title: Keep Your Dynamic Groups Compliant by Microsoft Graph Change Notifications and Azure Event Grid
subtitle:  Automatically Update Dynamic Group Membership Rules to Ensure Offboarding Users Are Removed
cover-img: /assets/img/GCN3.png
thumbnail-img: /assets/img/GCN1.png
share-img: /assets/img/GCN3.png
tags: [ Microsoft Graph, Azure Event Grid, Offboarding, Dynamic groups, Group governance, Access management ]

---
# Keep Your Groups Compliant: Automate Offboarding with Microsoft Graph and Azure Event Grid

## Table of Contents

- [Introduction](#introduction)
- [The Solution](#the-solution)
- [Architecture](#architecture)
  - [New Subscription for Microsoft Graph Change Notifications for Group Update](#new-subscription-for-microsoft-graph-change-notifications-for-group-update)
  - [Update the dynamic group membership rule](#update-the-dynamic-group-membership-rule)
- [Conclusion](#conclusion)

## Introduction

Group membership is a crucial aspect of Access Management in Entra ID and any IAM solution. In Entra ID, groups are used to manage access to applications, and resources, and assign roles. With the introduction of new **Entra ID Lifecycle Management** features, Microsoft has made it easier to manage group membership in Entra ID.

For example, as part of the Leaver Workflow, you can now perform the following actions on a user's group membership:

- Add the user to selected groups
- Remove the user from selected groups
- Remove the user from all groups

The last feature is particularly useful when a user leaves the organization. However, there is a small issue that needs to be addressed. The task **[Remove users from all groups](https://learn.microsoft.com/en-us/entra/id-governance/lifecycle-workflow-tasks#remove-users-from-all-groups)** only removes the user from Microsoft 365 and cloud security groups. It does not remove the user from Mail-enabled, Distribution, Dynamic, and Role-assignable groups.

To handle the removal of users from Mail-enabled, Distribution, and Role-assignable groups, you can create a [Custom Task Extension](https://learn.microsoft.com/en-us/entra/id-governance/lifecycle-workflow-extensibility). However, removing users from Dynamic groups is not possible as group membership is calculated based on a set of rules.

While this may not be a problem if your offboarding process already deactivates the user account and removes application access, it can affect audit and compliance reports. Level 1 auditors often rely on group membership information for these reports, and they may not have the necessary knowledge of application and resource access.

If you're thinking of using a dynamic query like as ```(user.accountEnabled -eq True)``` that will remove the offboarding user from the dynamic group as well and Lifecycle Workflow already has a built-in task to disable the user [Disable user account](https://learn.microsoft.com/en-us/entra/id-governance/lifecycle-workflow-tasks#disable-user-account), I partly agree with you. However, **Group Management** in any large organization is a delegated task, and as the IAM Architect you are often not in control of the dynamic group membership rules used while creating the groups. Well, not anymore! In this article, we will see how we can use Microsoft Graph Change Notifications and Azure Event Grid to keep dynamic groups compliant with the offboarding process by modifying the dynamic query for dynamic group membership rule.


## The Solution

The solution is simple. We will use Microsoft Graph Change Notifications to monitor the creation or updates of the dynamic groups in our tenant. Whenever a dynamic group is created or updated without the required query, we will update the query to include the ```(user.accountEnabled -eq True)``` condition. This will ensure that the offboarding user is removed from the dynamic group.

We will use Azure Event Grid to monitor the Microsoft Graph Change Notifications and Azure Automation or Azure Functions to update the dynamic group membership rule. I am not going to discuss the low-level implementation details of the solution. Instead, I will focus on the high-level architecture and the implementation steps.

## Architecture

![Architecture](/assets/img/GCN1.png)

The solution consists of the following components:

- We need to Authorize Microsoft Graph to create a partner event.
- Create a Microsoft Graph Subscription for Group Create and Update. Once the subscription is created with the notification URL, consisting of the Event Grid endpoint, it will create a partner topic in Event Grid.
- Activate the partner topic in Event Grid.
- Create an Azure Automation Runbook, Azure Logic App or Azure Function to update the dynamic group membership rule as an Event Handler for Event Grid.
- Subscribe to the events by creating an  Event subscription that uses the created Azure Automation Runbook, Azure Function, or Logic App.

### New Subscription for Microsoft Graph Change Notifications for Group Update

To get started, we will create a new subscription for Microsoft Graph Change Notifications for Group Update. We will use the following query to monitor the creation or updates of the dynamic groups in our tenant.

Here is the sample Microsoft Graph API call to create group change notification subscription.

```json
POST https://graph.microsoft.com/v1.0/subscriptions
{
            "changeType": "updated",
            "notificationUrl": "https://eo7dhjb8tfnh0qz.m.pipedream.net",
            "resource": "/groups",
            "expirationDateTime": "2024-02-17T08:08:58+00:00",
            "lifecycleNotificationUrl": "https://eo7dhjb8tfnh0qz.m.pipedream.net",
            "clientState": "e7ff5c2a-8f31-415f-8e21-6e548975a12c"
}
```

We can also use Powershell for the same.

```powershell
Connect-MgGraph
Connect-AzAccount

Import-Module Microsoft.Graph.ChangeNotifications

$subscriptionId = "e0f8145b-*********-ee65843b5555"
$resourceGroup = "EventGrid-RSG"
$partnerTopicName = "GroupChangeNotificationsUpdated"
$azureRegion = "northeurope"
$params = @{
 changeType = "created,updated"
 notificationUrl = "EventGrid:?azuresubscriptionid=$subscriptionId&resourcegroup=$resourceGroup&partnertopic=$partnerTopicName&location=$azureRegion"
 lifecycleNotificationUrl = "EventGrid:?azuresubscriptionid=$subscriptionId&resourcegroup=$resourceGroup&partnertopic=$partnerTopicName&location=$azureRegion"
 resource = "groups"
 expirationDateTime = [System.DateTime]::Parse("2024-01-19T18:23:45.9356913Z")
 clientState = "05a838f0-c8f4-4546-9316-98f9819d73ff"
}
$Subscription = New-MgSubscription -BodyParameter $params -Debug
```

Here is a sample change notification received by our Event Handler in my test tenant.

![Change Notification](/assets/img/GCN2.png)

### Update the dynamic group membership rule

Use the code snippet below to update the dynamic group membership rule.

```powershell

$groupId = "e0f8145b-*********-ee65843b5555" # We will get this from the event payload
$group = Get-MgGroup -GroupId $groupId -Property "id,membershipRule,groupTypes"
$group.MemberShipRule += " -and (user.Accoun
tEnabled -eq True)"
Update-MgGroup -GroupId $group.Id -MembershipRule $group.MembershipRule

```

You can also use the following Microsoft Graph API call to update the dynamic group membership rule.

```json
PATCH https://graph.microsoft.com/v1.0/groups/{id}
{
  "membershipRule": "string",
  
}
```

## Conclusion

In this article, we have seen how we can use Microsoft Graph Change Notifications and Azure Event Grid to keep dynamic groups compliant with the offboarding process by modifying the dynamic query for dynamic group membership rule.

This solution helps to ensure that offboarding users are removed from dynamic groups, even if those groups are explicitly created or updated by other users. This helps to maintain compliance with access management policies and ensures that only the correct users have access to resources.

I hope you found this article helpful. If you have any questions, please feel free to leave a comment below.

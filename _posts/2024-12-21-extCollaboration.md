
---
layout: post
title: External Collaboration Settings with Application Permissions
tags: [ EntraID , B2B , IAM ]

---
**External Collaboration Settings with Application Permissions**

Managing guest user invitations in Entra ID can be complex, especially when balancing collaboration needs with security policies. Here is an analysis of the **External Collaboration Settings** and their impact on invitations sent using application permissions.

**Option 1: No One in the Organization Can Invite Guest Users (Most Restrictive)**

This setting completely blocks all guest invitations, including those from administrators and applications. Attempting to invite users results in an error.

**Test Code:**

```powershell

$params = @{

invitedUserEmailAddress = "suryendubbhattacharyya@gmail.com"

inviteRedirectUrl = "https://myapp.contoso.com"

}

New-MgInvitation -BodyParameter $params

```

**Result:**

```

New-MgInvitation\_Create: Guest invitations not allowed for your company.

Contact your company administrator for more details.

```

This configuration ensures that no guest accounts can be created, enhancing organizational security.

**Option 2: Only Users Assigned to Specific Admin Roles Can Invite Guest Users**

With this setting, only users assigned roles such as **User Administrator** or **Guest Inviter** can send invitations. For application permissions, the app must **not** belong to the Guest Inviter role to respect the restriction.

**Test Code:**

```powershell

$params = @{

invitedUserEmailAddress = "suryendubbhattacharyya@gmail.com"

inviteRedirectUrl = "https://myapp.contoso.com"

}

New-MgInvitation -BodyParameter $params

```

**Result:**

```

Invitation Sent.

```

Applications with the correct permissions, such as User.Invite.All, can still send invitations under this configuration.

**Other Options: Less Restrictive Settings**

The remaining settings allow broader access, enabling most users and applications to invite guests. These options may not be ideal if strict control over guest user management is required.

1. Anyone in the organization can invite guest users including guests and non-admins (most inclusive)
2. Member users and users assigned to specific admin roles can invite guest users including guests with member permissions

**Conclusion**

To maintain control over guest invitations and ensure secure external collaboration, consider the following:

* Use the **"Only users assigned to specific admin roles can invite guest users"** setting.
* Exclude Users and Groups from the **Guest Inviter role**.
* Assign the User.Invite.All permission to application credentials.

This configuration balances security and functionality, enabling strict controlled guest access while adhering to organizational policies.



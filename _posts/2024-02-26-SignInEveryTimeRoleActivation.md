---
layout: post
title: Require Sign-in Every time a user activates role membership in Entra ID
subtitle:  Authentication Context Sign-in EveryTime
cover-img: /assets/img/RoleActivationLogo.png.jpg
thumbnail-img: /assets/img/RoleActivationLogo.png.jpg
share-img: /assets/img/RoleActivationLogo.png.jpg
tags: [ IAM, IGA , PIM, ENTRA, ENTRAID]
readtime: true

---
# Require Sign in every time a user activates role membership in Entra ID

One of the most common Privileged Identity Management (PIM) requirements is the ability to require sign-in every time a user activates a role membership and depending on the role permissions have a different level of secure authentication methods, like phishing-resistant security keys, passkeys or biometrics. In this article, we will show how you can configure Entra ID PIM to require sign-in every time a user activates a role membership.

## Concepts

![PIM Role Activation](/assets/img/RoleActivation.png)


## Add a new Authentication Context

First, we need to add a new authentication context that denotes the action of activating the role membership. We can also use an existing authentication context if we want to require it every time the user signs in for other sensitive actions like Conditional Access Policy modification or role assignment, that may need additional security.

1. Log in to the [Microsoft Entra admin center](https://entra.microsoft.com).
1. Navigate to the **Protection** > **Conditional Access** > **[Authentication Context](https://entra.microsoft.com/#view/Microsoft_AAD_ConditionalAccess/ConditionalAccessBlade/~/AuthenticationContext/fromNav/)** page.
1. On the Authentication Context page, click + New Authentication Context.
1. On the Add authentication context wizard, fill out the following information:
    * Name: Tier 0 Role Activation (Sign-in Everytime)
    * Description: Privilege Role Activation Requires Sign-In Every Time
    * Publish to apps: Checked - [x]

![Add Authentication Context](/assets/img/RoleActivation2.png)

## Configure the Privileged Identity Management settings for Entra ID Roles

1. Log in to the [Microsoft Entra admin center](https://entra.microsoft.com).
1. Navigate to the **Identity governance** > **[Privileged Identity Management](https://aka.ms/ad/pim)** page.
1. Select **Microsoft Entra Roles** under the Manage.
1. For this example Open Application Administrator role. settings.
1. On the Application Administrator page, choose **Role Settings**.
1. On the Role settings  – Application Administrator page, choose Edit.
1. On the Edit role setting – Application Administrator page, enable the On activation, require **Microsoft Entra ID Conditional Access authentication context** setting, and choose the **Tier 0 Role Activation (Sign-in Everytime)** authentication context created in the previous step.
1. Click Update.

![Role Settings](/assets/img/RoleActivation3.png)

## Create the Conditional Access Policy for Role Activation

1. Log in to the [Microsoft Entra admin center](https://entra.microsoft.com).
1. Navigate to the **Protection** > **Conditional Access** > **[Policies](https://entra.microsoft.com/#view/Microsoft_AAD_ConditionalAccess/ConditionalAccessBlade/~/Policies/fromNav/)** page.
1. Click the “+ New Policy” button to create a new policy.
1. Give your policy a name
    a. Name: Tier 0 Role Activation
1. Select **Users**

    a. You can select all users, specific users, or groups or exclude specific users or groups. For this example, we will select the member of our application administrator role.
    ![Select Users](/assets/img/RoleActivation4.png)


1. Select **Target resources**

    a. Set the `Select what this policy applies to` Authentication context.

    b. Select **Tier 0 Role Activation (Sign-in Everytime)** authentication context.
    ![Select Target Resources](/assets/img/RoleActivation5.png)

1. Select Session.

    a. Select Sign-in frequency.

    b. Select **Every time**.
    ![Select Session](/assets/img/RoleActivation6.png)

1. Set the policy status to On
Click “Create” to save your policy.



## End User Experience

When an admin activates the role membership for the role with the authentication context `Tier 0 Role Activation (Sign-in Everytime)` they are prompted to sign in again to complete the activation.

<center>
<video width="320" height="240" controls>
  <source src="/assets/img/RoleActivation8 (1).mp4" type="video/mp4">
</video>
</center>

## Summary

There is always some trade-offs between security and user experience. I feel re-authentication for activating high-privilege admin roles is a must-have. 

If the phishing phishing-resistant method can not be enforced due to the unavailability of physical security keys (passkey, 🗝️ 🔐 alternative is still not available), then the token can be replayed to activate PIM roles. 

As long as MFA claims are present in the token, Ca policy will not prompt for MFA, for eligible role activation in PIM, and even if we use a " require stronger authentication method ( of a non-phishing-resistant kind like Microsoft authenticator or another software-based authenticator app)", in Ca policy experience remains the same. 

By enforcing re-authentication and MFA ( or if possible, non-SMS-based MFA), We can reduce the vulnerability gap significantly, especially when rolling out the phishing-resistant MFA is not viable.



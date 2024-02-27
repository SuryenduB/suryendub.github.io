---
layout: post
title: Require Re-authentication for Registering Security Info (Adding Authentication Method) in Entra ID
cover-img: /assets/img/RoleActivationLogo.png.jpg
thumbnail-img: /assets/img/RoleActivationThumbnail.png
share-img: /assets/img/RoleActivationThumbnail.png
tags: [ IAM, IGA , ENTRA, ENTRAID , ConditionAccess , CA Policy]
readtime: true

---
# Require Re-authentication for Registering Security Info

Microsoft deserves commendation for its relentless efforts in pushing organizations to adopt **stronger authentication methods**. However, it is important to acknowledge that even with these measures in place, bad actors can still gain unauthorized access to employee accounts through tactics such as phishing and social engineering. Once inside, they can add authentication methods of their choice to bypass multi-factor authentication (MFA). This is why it is important to require users to re-authenticate every time they register their security information.

Fortunately, with the recent enhancement of Entra ID Authentication Context through Conditional Access Policies, organizations now can require users to re-authenticate every time they register their security information. This additional layer of security helps mitigate the risk of unauthorized access and reinforces the importance of regularly verifying user identities. By implementing this practice, organizations can further strengthen their security posture and protect sensitive information from falling into the wrong hands.

In this small article, I will show how you can configure Conditional Access Policies in Entra ID to require users to re-authenticate every time they register their security information.

## Configure the Conditional Access Policy for Registering Security Information

1. Log in to the [Microsoft Entra admin center](https://entra.microsoft.com).
1. Navigate to the **Protection** > **Conditional Access** > **[Policies](https://entra.microsoft.com/#view/Microsoft_AAD_ConditionalAccess/ConditionalAccessBlade/~/Policies/fromNav/)** page.
1. Click the “+ New Policy” button to create a new policy.
1. Give your policy a name
    a. Name: **CA301-Internals-BaseProtection-CombinedRegistration-Require-ReAuthentication**. (It is important to use a naming convention that is easy to understand and follow. I have used the **[Conditional Access for Zero Trust Resources
](https://docs.microsoft.com/en-us/azure/architecture/guide/security/conditional-access-zero-trust?msclkid=d1768a34ceda11ec9b6c8f244f8d05bd)** for naming the policy.)
1. Select **Users**

    a. You can select all users, specific users, or groups or exclude specific users or groups. For this example, we will select the member of our application administrator role.
    ![Select Users](/assets/img/RoleActivation10.png)

1. Select **Target resources**

    a. Set the `Select what this policy applies to` **User actions**.

    b. Select the checkbox **Register security information** for `Select the action this policy will apply to`.
    ![Select Target Resources](/assets/img/RoleActivation11.png)

1. Scroll Down and Select Session.

    a. Select Sign-in frequency.

    b. Select **Every time**.
    ![Select Session](/assets/img/RoleActivation12.png)

1. Set the policy status to On
Click “Create” to save your policy.


## End User Experience

When a user tries to register new security information, they will be prompted to re-authenticate.

<div style="display: flex; justify-content: center;">
  <video controls>
    <source src="/assets/img/RoleActivation9_muted.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>


## Summary

Requiring users to re-authenticate when registering security information adds an extra layer of security and helps prevent unauthorized access. This article explains how to configure Conditional Access Policies in Entra ID to implement this practice.



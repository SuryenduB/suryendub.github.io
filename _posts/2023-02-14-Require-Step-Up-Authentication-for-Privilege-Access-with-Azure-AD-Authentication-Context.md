---
layout: post
title: Require Step Up Authentication for Privilege Access with Azure AD Authentication Context
subtitle: Don't leave your Azure AD environment vulnerable - Take action to secure it now!
cover-img: /assets/img/PIM-Role-Assignment-Nestor.jpg
thumbnail-img:  /assets/img/PIM.jpg
share-img: /assets/img/PIM-Role-Setting-Owner.jpg
tags: [Azure, Azure AD, security, MFA,PIM, Privileged Identity Management]
---


# Tutorial : Require Step Up Authentication for Privilege Access with Azure AD Authentication Context

To enhance the security of your administrators, it is recommended to enforce the use of multifactor authentication (MFA or 2FA). Multifactor authentication adds an additional layer of security by requiring users to provide multiple authentication factors, This approach enhances security by requiring users to provide multiple authentication factors, thereby lowering the risk of an attack through compromised passwords. 

You have the option to configure MFA challenges during user sign-in or role activation in Privileged Identity Management (PIM) in Azure Active Directory (Azure AD), a component of Microsoft Entra. This means that if a user fails to complete MFA during sign-in, PIM will prompt them to complete it. By implementing these measures, you can enhance the security of your Azure environment and protect it against potential security breaches.

## **Story so far : Azure MFA and PIM**

You can ensure administrators need to perform MFA,   at the time of activating eligible roles from Azure PIM Console.
You need to configure the following setting [On activation, require multi-factor authentication](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-resource-roles-configure-role-settings#on-activation-require-multi-factor-authentication).

![Role Settings](/assets/img/PIM-Role-Setting-Contributor.jpg)

This will ensure that eligible admins will be prompted for MFA at the time of Activating his account.

However, Privileged Identity Management does not enforce multifactor authentication when the user uses their role assignment if they complete Multi-Factor-Authentication at the time of  signing in Azure Portal i.e. they have provided multi-factor authentication earlier in the same session. 

You can read more in detail about [MFA and PIM](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-how-to-require-mfa). The rationale behind this is to reduce MFA fatigue among administrators who may find it burdensome to undergo multiple MFA prompts within a short period of time. For instance, an attacker could hijack the session of a signed-in Azure Admin and request privilege role activation, without prompting the admin for MFA as they have already completed it earlier during this session. 

One more use case  for Authentication Context is to enforce stringent authentication requirements for specific roles. For example , while eligible admin can activate contributor role from any location , activating the Owner role may require the user to sign in from a trusted location.

## **The solution**

 1. Create **Authentication Strength** in Azure AD Portal.
 1. Create **Authentication Context** for PIM.

 2. Create **Conditional Access policy** that would enforce **Authentication Strength** for this **authentication context**.

 3. Select the **Authentication Context** in the PIM settings for the respective roles.
 
### **Create Authentication Strength in Azure AD Portal.**
You  need to create a custom Authentication Strength. This is required to chose a different authentication method from one required for sign-in. In this tutorial , you need to  set  **SMS** single factor authentication, as the authentication method. 
>Note:  Users in our tenant need to perform MFA  using **Microsoft Authenticator** app push notification.

1. Navigate to Azure Active Directory > Security > Authentication methods > Authentication strengths (Preview).
2. Select New authentication strength.

3. Add a descriptive name : "PIM Authentication Strength"

4. Optionally provide a Description : "PIM Authentication Strength SMS Verification "

5. Scroll down and select Single factor authentication **SMS**.

Choose Next and review the policy configuration. 

![New-Authentication-Policy](/assets/img/PIM-CA-Authentication-Strength.jpg)

### **Create Authentication Context for PIM.**

1. Navigate to **Azure Active Directory > Security > Conditional Access > Authentication context.**
2. Create new authentication context definitions by selecting New authentication context in the Azure portal. Organizations are limited to a total of 25 authentication context definitions.
3. Enter value for following attributes: **Diplayname** , **Description** , **Publish to apps** , **Id**.  
    ![New-Authentication-Context](/assets/img/PIM-CA-Authentication-Context.jpg)

### **Create Conditional Access policy that would enforce previously created Authentication Strength requirements for this Authentication context.**
Next you need to create Conditional Access policy that would enforce requirements for this authentication context.

1. Browse to Azure Active Directory > Security > Conditional Access
2. Select New policy.
3. Give your policy a name : PIM Role Policy to trigger MFA Prompt
4. Under Assignments, select Users or Group and select the Eligible administrators for PIM Roles.
5. Administrators can select published authentication contexts in their Conditional Access policies under Assignments > Cloud apps or actions and selecting Authentication context from the Select what this policy applies to menu.

    ![CA-Policy](/assets/img/PIM-CA-Policy.jpg)

6. Select Under Access controls > Grant, select Grant access, In this case Select Require Authentication Strength and add the custom Authentication Strength created before from the drop down menu .
Confirm your settings and set Enable policy On.

   ![CA-Policy-Authentication-Strength](/assets/img/PIM-CA-Policy-Authentication-Strength.jpg)

7. Select Create to create to enable your policy.

### **Add the Authentication Context in the PIM settings for the respective role.**
Navigate to Azure AD Privileged Identity Management -> Azure Resources -> Select the Resource -> Select Setting. 
 Modify following setting  **On activation , required**.
    Select  **Authentication Context** for Owner role and **Azure MFA** for Contributor role.

![Owner-Role-Setting](/assets/img/PIM-Role-Setting-Owner.jpg)
![Contributor-Role-Setting](/assets/img/PIM-Role-Setting-Contributor.jpg)

## **Test the Solution**

### **Activate Owner Role**
1. Now open a private browsing session and  log in to Azure Portal as **Eligible Administrator** . User need to perform MFA at the time of Sign in. You can configure the Conditional access policy to [Require MFA for all users ](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/howto-conditional-access-policy-all-users-mfa).
2. Browse to **Azure AD Privileged Identity Management>My Roles>Select Azure resource roles**

    ![Eligible Roles](/assets/img/PIM-Role-Activate.jpg)

3. Click on Activate under Action for Owner Role. You will notice the warning 
> **Warning**: A Conditional Access policy is enabled and may require additional verification.Click to continue

![Activate-Owner](/assets/img/PIM-Role-Activate-Owner.jpg)

4. Click on the warning and user will be prompted for verify using SMS

![Activate-Owner-MFA](/assets/img/PIM-Role-Activate-Owner-MFA.jpg)

### **Activate Contributor Role**

1. Follow the  step 1-2 from previous section.
2. Click on Activate under Action for Contributor Role.
3. Notice user will not be prompted for MFA.


![Activate-Contributor](/assets/img/PIM-Role-Activate-Contributor.jpg)

In conclusion, enforcing multifactor authentication for privilege access in Azure AD can significantly enhance the security of your environment by adding an additional layer of security to user authentication. However, using **Azure MFA** at the time of role activation may not be sufficient to protect admin users.

Azure PIM will not prompt MFA for eligible admin, if they have provided multi-factor authentication earlier in the same session.

To address these concerns, the tutorial recommends creating an Authentication Strength and Authentication Context for PIM, which can enforce stringent authentication requirements for specific roles. By adding these security measures to the PIM settings for the respective role, you can ensure that only eligible administrators are authorized to access privileged resources only after performing required stringent authentication.  Overall, this tutorial provides a practical guide for implementing these security measures in Azure AD and highlights the importance of strong authentication and access control measures to protect against potential security breaches.







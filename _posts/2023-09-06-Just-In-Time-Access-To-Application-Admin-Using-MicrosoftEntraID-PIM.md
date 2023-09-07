---
layout: post
title: Just In Time Application Administration Using PIM in Microsoft Entra ID - A  Step By Step Guide to Improving Security Posture
subtitle :  Reduce the Risk of Unauthorized Access with JITAA
cover-img: /assets/img/PIMForApp22.jpg
thumbnail-img: /assets/img/PIMForApp22.jpg
share-img: /assets/img/PIMForApp.jpg
tags: [ Azure Active Directory, EntraID, MicrosoftEntra, Security, ZeroTrust, PrivilegedIdentityManagement, JIT]

---
# Just In Time Application Administration Using PIM in Microsoft Entra ID - A  Step By Step Guide to Improving Security Posture

One of the critical pillars in the implementation of Zero Trust Security using Azure Security Technologies is the **Privileged Identity Management** capability. Within this framework, two key components stand out: **Just In Time Administration (JIT)** and enforcing **Multi-Factor Authentication (MFA)** at the time of actual role activation.

JIT Administration ensures that privileged access is granted only when needed, minimizing the window of vulnerability and reducing the attack surface. By enforcing MFA during role activation, Azure adds an extra layer of security, ensuring that even if a malicious actor gains access to privileged accounts, they are thwarted by the added authentication step. This approach not only aligns with the principles of least privilege but also bolsters the overall security posture, making it significantly more challenging for unauthorized users to compromise sensitive systems and data.

Recently, Microsoft unveiled a Preview of the **Entra ID** capability, marking an exciting development in the realm of identity and access management. With this innovative feature, organizations can now explore the possibility of extending **PIM** functionality to other SCIM (System for Cross-domain Identity Management) supported Line of Business (LOB) applications. This advancement signifies Microsoft's commitment to providing scalable and adaptable solutions for identity management, allowing businesses to streamline their operations and enhance security not only within the Microsoft ecosystem but also across a wider array of third-party applications, ultimately promoting a more efficient and secure identity governance framework.

Recently, Microsoft unveiled a Preview of the **Entra ID** capability, marking an exciting development in the realm of identity and access management. With this innovative feature, organizations can now explore the possibility of extending its functionality to other SCIM (System for Cross-domain Identity Management) supported Line of Business (LOB) applications. This advancement signifies Microsoft's commitment to providing scalable and adaptable solutions for identity management, allowing businesses to streamline their operations and enhance security not only within the Microsoft ecosystem but also across a wider array of third-party applications, ultimately promoting a more efficient and secure identity governance framework.

In this article, we will delve deeper into the exciting possibilities that organizations can effectively implement to harness their potential to extend the **Zero Trust** capability.


## Implementation of JIT for SCIM-Based Application Privilege Access

Certainly, here are the step-by-step instructions for implementing Privileged Identity Management (PIM) for an application, based on the provided diagram:

![PIM for App SequentialSteps](/assets/img/PIMForApp.jpg)

Let me explain what we are trying to accomplish here. The Organization **SuryenduB** wants to implement this capability in one of the Widely used Applications **App1**. The application supports Users and Group Provisioning using **SCIM** protocol and is configured for  **Single Sign On** using EntraID as an Identity Provider. Administrator access permissions are assigned to **Access Group**  within the Application **App1**.

Users who need Access to the Application **App1** are added to **Group X** for normal access.

**Direct Assignment**: User A is directly assigned to **Group X**, indicating that User A has a direct membership in Group X.

![Direct Assignment to Group A](/assets/img/PIMForApp1.jpg)

**Eligible Assignment**: UserA is eligible for assignment to **Group Y**, but this assignment is not active yet. User will need to [activate their group membership](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/groups-activate-roles#activate-a-role)  in **Group Y**.

![Eligible Assignment Group B](/assets/img/PIMForApp2.jpg)

**Assign Group X to App1**: Group X is assigned to App1, meaning that the members of Group X will be created in App1 whenever the Provisioning Job Runs, and they can access Application X by Single Sign On (SSO).

![Application Group Assignment](/assets/img/PIMForApp3.jpg)

**Assign Group Y to App1**: GroupY is assigned to App1, Provisioning Job will create this Group in **App1**. This group will be assigned Privilege Administration Access in **App1**.

![Application Admin Group Assignment](/assets/img/PIMForApp4.jpg)

**Provision User A to App1**: The provisioning workflow initiates the process of provisioning UserA to App1, granting UserA access to App1. App1 is enabled for SCIM Based Provisioning.

![Provisioning Configuration in App1](/assets/img/PIMForApp5.jpg)

![User A Provisioned in App1](/assets/img/PIMForApp8.jpg)

**Provision Group X to App1**: The provisioning workflow also provisions GroupX to App1.

![Group X Provisioned in App1](/assets/img/PIMForApp6.jpg)

**Provision Group Y to App1**: Similar to GroupX, Group Y is provisioned to App1, ensuring that its members have the required access and roles in App1.

We can verify all the Provisioning activities in Provisioning Logs.

![Group Y Provisioned in App1](/assets/img/PIMForApp7.jpg)

As Evident from Group Y, currently Group Y does not have any members in App 1.

![Provisioning of  Users and Groups In App1 Provisioning Logs](/assets/img/PIMForApp9.jpg)

**Admin Role**: **Group Y** is granted an administrative role in **App1**, which might include elevated privileges or responsibilities within the applications.

**Activate Group Membership Group Y**: User A activates their membership in GroupY, indicating that they want to be part of this group and have associated privileges.

1. ![User A Role Activation in PIM  ](/assets/img/PIMForApp10.jpg)

2. ![User A Role Activation in PIM  ](/assets/img/PIMForApp11.jpg)

**Add User A to Group Y**: The PIM system adds **User A** to **Group Y**, ensuring that UserA is now a member of this group in **EntraID**.

 ![User A Role Activation in PIM : Assignment Time ](/assets/img/PIMForApp12.jpg)

 Pay close attention to the PIM Audit Log and note the role Activation time.

 ![User A Role Activation in PIM Audit Log](/assets/img/PIMForApp13.jpg).

**Runs Provisioning Job Immediately**: The provisioning workflow runs a **synchronization job** immediately to update the group membership of Group Y in App1, ensuring that UserA's privileges are correctly reflected in **App1** as **Admin Roles are Assigned to Group Y in previous step**.

Navigate to **Enterprise applications > App1 > Provisioning > Provisioning Logs** to verify that the provisioning Job has run immediately.

![Group Y Provisioning in Provisioning Log](/assets/img/PIMForApp14.jpg).

We can verify Group Membership is updated in Application **App1**.

![Group Y Provisioning in Provisioning Log](/assets/img/PIMForApp15.jpg).

ID: 6f25ed20-5d31-4737-938a-eb8727bf8d01 is the ID of User A in App 1.
User A can sign in to **App1** and perform administrative tasks.

![User A Details in App1](/assets/img/PIMForApp16.jpg)

**Deactivate Membership / Time Expired**: User A decides to deactivate their membership in GroupY, or their membership expires due to time constraints.

![User A Details in App1](/assets/img/PIMForApp17.jpg)

**Remove User A from Group Y**: The PIM system removes User A from GroupY in **EntraID**.

![User A Details in App1](/assets/img/PIMForApp18.jpg)

**Runs Provisioning Job Immediately**: Once The provisioning workflow runs synchronization Job  to update the group membership of **Group Y** in **App1**.

![Group Y Provisioning Log Modification](/assets/img/PIMForApp19.jpg)

![Group Y Provisioning Log Modification](/assets/img/PIMForApp20.jpg)

**Admin Role Revoked from User A**: As a result of UserA's deactivation or membership expiration, the admin role is revoked from UserA in **App1**, removing any associated privileges.

These instructions outline the actions and processes for implementing Privileged Identity Management (PIM) for an application.

![Group Membership Removal in App1](/assets/img/PIMForApp21.jpg)

## Conclusion

Just In Time Application Administration for Line of Business Apps or **Managed Cloud Systems** like **GCP**, **AWS**, using PIM is a powerful tool that can help organizations improve their security posture and reduce the risk of unauthorized access to sensitive systems and data. Granting privileged access only when needed and requiring MFA at the time of activation, can help minimize the window of vulnerability and make it more difficult for attackers to compromise systems.

The implementation of JITAA using PIM is simple, but the benefits are significant. By following the steps outlined in this article, organizations can get started with JITAA and begin to reap the rewards of improved security.

**Key takeaways:**

* JIT Access is a key component of Zero Trust security.
* JIT Application Admin Access can help to reduce the risk of unauthorized access to sensitive systems and data.
* JIT Application  can be implemented using PIM.


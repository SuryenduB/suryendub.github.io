---
layout: post
title: Securing Legacy Applications with Entra Private Access and Conditional Access
subtitle : Zero Trust Network Access with Entra ID Private Access
cover-img: /assets/img/Private%20Access%20To%20LDAP.jpg
thumbnail-img: /assets/img/Private%20Access%20To%20LDAP.jpg
share-img: /assets/img/Private%20Access%20To%20LDAP.jpg
tags: [ Azure Active Directory, EntraID, Networking, ZeroTrust, Security]

---

# Securing Legacy Applications with Entra Private Access and Conditional Access

**Legacy applications** are those that use older, less secure authentication protocols, such as Basic authentication, LDAP, etc. These protocols are vulnerable to cyberattacks, such as password guessing and man-in-the-middle attacks. Even SMB file share or **Remote Windows or Linux Servers** used within your organization can be considered as a **Legacy Application**.  

New **Microsoft Entra Private Access** and **Conditional Access** are two security features that can be used to secure legacy applications.

**Entra Private Access** uses a ZTNA architecture to verify the identity of the user and device before granting access to the application. This helps to prevent unauthorized access to the application.

**Zero Trust Network Access** is a security approach that assumes no user or device is trusted by default. Access to applications and data is granted on a need-to-know basis, and only after the user or device has been authenticated and authorized.

* **Entra Private Access uses Azure AD Application Proxy connector to broker the connection between the service and the internal resource.** Global Secure Client installed on user devices is used to connect to Internal Resources.  It use the ZTNA architecture model to verify the identity of the user and device before granting access to the private application using **Conditional Access**.

* **Conditional Access**  can be used to enhance the security of legacy applications by requiring users to authenticate using multi-factor authentication (MFA), connect from compliant devices, and use phishing-resistant MFA methods. This helps to ensure that only authorized users can access the applications, and that they are doing so from secure devices.

![Alt text](/assets/img/Private%20Access%20To%20LDAP.jpg)

## Implementation Steps

1. **Install the Application Proxy Connector in your on-premises network.** The Application Proxy Connector is a software appliance that you install in your on-premises network. It creates a secure tunnel between your on-premises network and the Entra Private Access service.

2. **Enable Microsoft Entra Private Access.** Once you have installed the Application Proxy Connector, you need to enable Microsoft Entra Private Access. This can be done in the Microsoft Entra admin center. Once Microsoft Entra Private Access is enabled, you can start configuring your access policies.

3. **Download and install the Global Secure Access client on end-user devices.** The Global Secure Access client is a software application that you install on end-user devices. It allows users to connect to your private applications securely. The Global Secure Access client uses the secure tunnel created by the Application Proxy Connector to connect to your private applications and internal resources.

4. **Configure Quick Access or Per App Access.**

    * Quick Access is a simple way to secure all of your private applications with a single policy.

    * Per App Access allows you to create more granular policies for specific applications.

    You need to add application segmentation to define the FQDNs and IP addresses that you want to include in the traffic for Microsoft Entra Private Access. Assign users and groups that need access to the private applications.

5. **Implement  Conditional Access policy**: Crete Conditional Access Policy for QuickAccess App or Any Private Apps created in step 4.

## Conclusion

Legacy applications are a common target for cyberattacks. By combining Entra Private Access and Conditional Access, organizations can protect their legacy applications from cyberattacks and ensure that only authorized users can access them. For Example, with Entra Private Access, users must complete multi-factor authentication (MFA) to connect to a server with RDP. Similarly, any user authenticating with a legacy LDAP directory must also complete MFA, providing an extra layer of verification that was not possible before.

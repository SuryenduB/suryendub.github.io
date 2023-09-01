---
layout: post
title: Entra ID Alternate Login ID A Useful Feature with Some Potential Pitfalls
subtitle: Understanding the Impact of Entra ID Alternate Login ID on OAuth/OIDC Tokens
cover-img: /assets/img/ProxyAddress6.jpg
thumbnail-img: /assets/img/ProxyAddress6.jpg
share-img: /assets/img/ProxyAddress6.jpg
tags: [ Azure Active Directory, EntraID, Networking, ZeroTrust, Security]

---

# Entra ID Alternate Login ID: A Useful Feature with Some Potential Pitfalls

## What is Entra ID Alternate Login ID?

In Entra ID, an alternate login ID is like an optional way to log in using a Proxy Address. It's a different ID that's not your usual username (UPN). This can be really helpful when your usual ID is hard to remember or when, due to business or compliance reasons, the organization doesn't want to use the on-premises UPN to sign in to Entra ID.

For instance, consider a merger scenario in which users from a newly onboarded organization are integrated into the tenant of the acquiring company. The acquiring company may opt to consolidate all user accounts into their on-premises Active Directory while retaining their established company User Principal Name suffix (such as BigCompany.com). Concurrently, they may wish to incorporate email addresses from the merged company as email aliases (e.g., SmallCompany.com). To enable this, these email aliases must be formally added and verified as custom domains in Entra ID. To make this work, they also need to verify Custom Domain SmallCompany.com in the acquired company's Entra ID Tenant.

The key advantage here is that by configuring the Alternate Login as the designated alternate login ID, users from the merged organization can seamlessly log in using their familiar credentials, ensuring a smoother transition and user experience during the merger process.
![Login with Email or UPN](/assets/img/ProxyAddress6.jpg)

## Powershell Code for Configuring Alternate Login ID

```powershell
# Install the Microsoft.Graph module and connect to the Microsoft Graph API
Install-Module Microsoft.Graph
Connect-MgGraph -Scopes "Policy.ReadWrite.ApplicationConfiguration" -TenantId organizations

# Define the Entra ID policy definition
$AzureADPolicyDefinition = @(
  @{
     "HomeRealmDiscoveryPolicy" = @{
        "AlternateIdLogin" = @{
           "Enabled" = $true
        }
     }
  } | ConvertTo-JSON -Compress
)

# Define the Entra ID policy parameters
$AzureADPolicyParameters = @{
  Definition            = $AzureADPolicyDefinition
  DisplayName           = "BasicAutoAccelerationPolicy"
  AdditionalProperties  = @{ IsOrganizationDefault = $true }
}

# Create the Entra ID policy
New-MgPolicyHomeRealmDiscoveryPolicy @AzureADPolicyParameters
```

## Test alternate login

You can set up an Entra ID application to test OAuth2.0 easily. This helps you troubleshoot custom OAuth/OIDC token claims. Here's how to do it:

1. Go to your Entra ID portal and find "App registrations." Click on "New registration."

2. Name your application and choose the account types you want to support. For the Redirect URI, select "Web" and type in "[https://jwt.ms](https://jwt.ms)". Then, register your application. ![App Config 1](/assets/img/ProxyAddress2.jpg)

3. After registration, go to the Authentication menu. Enable the "Implicit grant flow" for Access and ID tokens. Also, allow the public client flow. Save these changes. ![App Config 1](/assets/img/ProxyAddress3.jpg)

4. Now, you need to create an application access URL. Use the following template (it should be in one line, but it's broken into lines here for readability):

```HTTP
https://login.microsoftonline.com/<tenantName or ID>/oauth2/v2.0/authorize
?client_id=<AppID of the JWT app you just created>
&nonce=defaultNonce
&redirect_uri=https%3A%2F%2Fjwt.ms
&scope=<the scopes, separated by + , example openid+profile>
&response_type=<id_token or token>
&prompt=<login or consent> (optional, use login to force re-auth, or consent to force consent>
```

For ID Token

```url
https://login.microsoftonline.com/XXXXXXXX-c5c4-4088-88cc-2f799eXXXXXX/oauth2/v2.0/authorize?client_id=xxxxxe391-5e42-44a2-93c4-f55867c08xxxxf&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid+profile+User.read&response_type=id_token
```

For Access Token

```url
https://login.microsoftonline.com/XXXXXXXX-c5c4-4088-88cc-2f799eXXXXXX/oauth2/v2.0/authorize?client_id=xxxxxe391-5e42-44a2-93c4-f55867c08xxxxf&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid+profile+User.read&response_type=token
```

This setup allows you to test OAuth2.0 and troubleshoot token claims for your Entra ID SaaS Application and [https://jwt.ms](https://jwt.ms).

## Issue with preferred username and unique name claim

App developers can use the `preferred_username` or `unique_name`  claims , which is a standard claim in OpenID Connect (OIDC), which is an authentication layer built on top of OAuth 2.0. 

The `preferred_username` or `unique_name`  claim typically represents the preferred or primary username of the authenticated user. It can be useful for applications that want to display a user's primary username or identifier, especially in scenarios where multiple identifiers or email addresses may be associated with the user account.

Here are some common use cases for the `preferred_username` or `unique_name` claim:

1. **User Identification:** Applications can use the `preferred_username` or `unique_name` claim to identify and display the primary username or email address associated with the user's account. This can help ensure a consistent and recognizable user experience.

2. **User Profile:** The claim can be used to populate user profile information within the application, making it easier to personalize the user's experience.

3. **Logging and Auditing:** Some applications may use the `preferred_username` or `unique_name` claim for logging and auditing purposes to track user activity and interactions.

During my test, I performed two separate login scenarios using the UPN and Email credentials. I observed a significant difference in the Token returned, specifically in the values of the 'preferred_username' claim in ID Token and 'unique_name' in the access token. I've included screenshots to illustrate this:

User Details:

- UPN : Brian.Cox@XXXXXXXX.onmicrosoft.com
- Email : brian.cox@smallcompany.com

Case 1:
![Login With UPN](/assets/img/ProxyAddress4.jpg)

Case 2:
![Login with Email](/assets/img/ProxyAddress5.jpg)

In this case Applications using the claims may failed to identify the user , when user logs in using Alternate IDs.Application developer may consider using other claim attributes.

It's worth noting that this aligns with the guidance provided in Microsoft Documentation. According to it, when a user signs in with a non-UPN email, the 'unique_name' and 'preferred_username' claims, if present in the ID token, will reflect the non-UPN email.

## Conclusion

In conclusion, Entra ID Alternate Login ID is a useful feature that can be used to allow users to sign in to Azure AD using an identifier that they are more likely to remember. However, it is important to be aware of the potential impact on the preferred_username and unique_name claims in OAuth/OIDC tokens. If applications are relying on these claims to identify users, they may need to be updated to use other claim attributes.

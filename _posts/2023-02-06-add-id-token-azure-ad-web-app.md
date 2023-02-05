---
layout: post
title: Enable ID Tokens from Application Manifest
subtitle: Enable the identity platform to issue ID tokens for your app
tags: [AzureAD , Oauth]
---

# Enable ID Tokens from Application Manifest

To enable the identity platform to issue ID tokens for your app, you need to enable the hybrid flow. The hybrid flow combines the use of the authorization code grant for obtaining access tokens and OpenID Connect (OIDC) for getting ID tokens.


## App Settings
Here is a snippet from the appsettings.json file of a web application:
```json
"AzureAd": {
    "Instance": "https://login.microsoftonline.com",
    "TenantId": "4d5f18ee-5b52-4315-85aa-******",
    "ClientId": "826de1d0-269c-41c3-bcbf-******",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath": "/signout-oidc"
  }
```
## Steps to Enable ID Tokens

1. Navigate to your app registration in the Azure portal and select the application.
2. In the Manage section, select the `manifest`.
3. Modify the following setting:
   ```json
   "oauth2AllowImplicitFlow": true
   ```
4. In the same manifest, modify the replyUrlsWithType array as follows:
   ```json
   "replyUrlsWithType": [{
       "url": "https://localhost:7046/signin-oidc",
       "type": "web"
    }]

        


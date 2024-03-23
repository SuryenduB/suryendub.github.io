# Managing Claim Mapping Policies with Microsoft Graph API

Identity and Access Management (IAM) Engineer, I grapple with a crucial challenge: securely onboarding applications into an organization’s identity system.. This is a critical task because it ensures that the organization's applications are secure and compliant with the organization's security policies.

In a previous article, I delved into the process of securely onboarding applications using **Terraform**. You can find that piece here: [Securely Onboarding Applications with Terraform](https://suryendub.github.io/2024-01-30-terraform-azuread-application/).

Recently, I came across an insightful blog post by one of my favorite Microsoft Identity bloggers, [Eric Woodruff](https://www.linkedin.com/in/ericonidentity/), In his article titled "**Spam Spoofing Users in Azure AD with SAML Claims Transformations**", Eric highlights a critical point: SAML claims management in Entra ID is not inherently secure.

In this blogspot, I will show you how to use claims mapping policies to manage how you can securely manage application claims and take one more step towards **Git-Ops** management model of IAM from the error-prone manual Click-Ops process and have a robust security posture.

## What are Claims Mapping Policies?

A claims mapping policy  object represents a set of rules enforced on individual applications or on all applications within an organization that modifies the claims included in tokens issued to the application.

## Creating a new Claims Mapping Policy

Let us first create a Claim Mapping policy for SAML application that send basic claims to the application.

```powershell
Import-Module Microsoft.Graph.Identity.SignIns

$params = @{
 definition = @(
  '{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true"}}'
 )
 displayName = "New Saml Claims Policy"
}

$policy = New-MgPolicyClaimMappingPolicy -BodyParameter $params
```

Now that we have create a new Claims Mapping Policy we need to assign it to our application.
I have previously written about ADFS Claims X-Ray tool that can be used to see the claims that are being sent to the application. Let us use the same application to see the claims that are being sent to the application.

First we need to retrieve the service principal ID of the application.

```powershell
$servicePrincipalId = Get-MgServicePrincipal -Filter "DisplayName eq 'Entra Claims XRay'" | Select -ExpandProperty Id
```

We will now assign the Claims Mapping Policy to the application.

```powershell
$claimsMappingPolicyId = $policy.Id
$params = @{
 "@odata.id" = "https://graph.microsoft.com/v1.0/policies/claimsMappingPolicies/$claimsMappingPolicyId"
}
$assignment = New-MgServicePrincipalClaimMappingPolicyByRef -ServicePrincipalId $servicePrincipalId -BodyParameter $params

```

Now that we have assigned the Claims Mapping Policy to the application, we can now see the claims that are being sent to the application.

![Basic Claims](.\"C:\Users\14784425548699500622\Desktop\Assets\Img\BasicClaims.png")

## Update the Claims Mapping Policy to include additional claims

We have seen the basic claims that are being sent to the application, let us update the Claims Mapping Policy to include additional claims. In this example, we will add a new SAML claims to the application. In this example, we will add a new claim `country` to the application. This will send the country of the Entra ID Tenant to the application.

```powershell
$params = @{
 definition = @(
  '{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true","ClaimsSchema":[{"Source":"company","ID":"tenantcountry","SamlClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/country"}]}}'
 )
 
}

Update-MgPolicyClaimMappingPolicy -ClaimsMappingPolicyId $claimsMappingPolicyId -BodyParameter $params


```

![Country Claims](.\"C:\Users\14784425548699500622\Desktop\Assets\Img\CountryClaims.png")

## Send Group Claims to the Applications using Claims Transformation

In the previous example, we saw how to send basic claims and additional claims to the application. In this example, we will see how to send group claims to the application. In order to send group displayname as claims, to the application we have to update the application manifest file.

```powershell

$params = @{
 definition = @(
  '{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true","ClaimsSchema":[{"Source":"company","ID":"tenantcountry","SamlClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/country"}]}}'
 )
 
}

Update-MgPolicyClaimMappingPolicy -ClaimsMappingPolicyId $claimsMappingPolicyId -BodyParameter $params

```powershell



- Application Manifest
```json
"acceptMappedClaims": true,
"saml2Token": [
   {
    "name": "groups",
    "source": null,
    "essential": false,
    "additionalProperties": [
     "cloud_displayname"
    ]
   }
  ]
```json

We can verify that the group claims are being sent to the application using the ADFS Claims X-Ray tool.

![Group Claims](.\"C:\Users\14784425548699500622\Desktop\Assets\Img\GroupClaims.png")

## Transforming Claims using Claims Transformation

In this example, we will see how to use claims Transformation to transform the group claims that are being sent to the application. We will also see how we can use claims Transformation to apply regex replacement to the group claims that are being sent to the application.
Many applications require Role claims to be sent to the application. We can use the same method to create a new Claims Mapping Policy that sends Role claims to the application after transforming the user group details.

```powershell

```powershell
Update-MgPolicyClaimMappingPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema":[{"Source":"user","ID":"groups"},{"Source":"transformation","ID":"ReplaceMail","TransformationId":"RegexReplaceTransform","SamlClaimType":"http://schemas.microsoft.com/ws/2008/06/identity/claims/role"}],"ClaimsTransformations":[{"ID":"RegexReplaceTransform","TransformationMethod":"RegexReplace","InputClaims":[{"ClaimTypeReferenceId":"groups","TransformationClaimType":"sourceClaim","TreatAsMultiValue":"true"}], "InputParameters": [{"ID":"regex","Value":"acl_(?<group>\\S{2})_.*"},{"ID":"replacement","Value":"{group}_Role"}],"OutputClaims":[{"ClaimTypeReferenceId":"ReplaceMail","TransformationClaimType":"outputClaim"}]}]}}') -DisplayName "TransformClaimsExample2" -ClaimsMappingPolicyId $claimsMappingPolicyId
```

![Transformed Claims](.\"C:\Users\14784425548699500622\Desktop\Assets\Img\TransformedRoleClaims.png")



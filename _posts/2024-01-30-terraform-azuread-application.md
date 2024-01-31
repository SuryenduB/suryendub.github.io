---
layout: post
title: Prevent Overpriviliged Application Creation and tie it to Identity Governance in Entra ID
subtitle:  Streamline Azure AD App Onboarding with Terraform  A Step-by-Step Guide
cover-img: /assets/img/terramod1.png
thumbnail-img: /assets/img/terramod2.png
share-img: /assets/img/terramod1.png
tags: [ Terraform,Entra ID,Application Registration,Access management,Access packages,Identity Governance,RBAC,Catalog
Application Management ]

---

# Prevent Overpriviliged Application Creation and tie it to Identity Governance in Entra ID

## Why

As an IAM engineer, a few things give me sleepless nights quite like the idea of a developer creating an application in Azure AD and granting it permissions to read and write all users and groups in the directory. Or worse, not having a semblance of an RBAC process and assigning all users to the application with the Admin Role. If you've ever spent hours trying to restrict a developer from adding overprivileged permissions to an application, raise a glass for me. Developers, bless their talented and focused souls, can sometimes be a bit stubborn when it comes to security. Let's just say they have other priorities, like crafting amazing code. Cheers to the developers, the true heroes of the digital realm!

![Flow Diagram for Terraform Module](/assets/img/terramod1.png)

In a Just World, it would be left to gritty IAM engineers like you and me to decide how applications are created in the tenant and what permissions they are granted. But in the real world, we have to work with developers and other stakeholders to ensure that applications are created quickly and efficiently without the bureaucratic red tape that sometimes bogs down the process. Isn't life all about finding the right balance? Grappling with this dilemma, I came up with a solution that I think is a good compromise between security and agility.

What if we strip the application developers of the right to create applications directly in the directory and allow them to create the same in Terraform, where we can enforce basic application security that is already in place? This way, we can ensure that the applications are created securely and the developers can still create applications without having to wait for the IAM team to create them. It also brings into it the gitops process workflow along with it to ensure, there is always an IAM engineer approving the new entrant in the directory. After all, we all know that developers have a knack for crafting amazing code and sometimes security just needs a little nudge in the right direction. So here's to finding that perfect balance between security and developer agility, because in the end, we're all in this together!

## How

 I spent my entire weekend coding a  [Terraform module](https://github.com/SuryenduB/terraform-azuread-application) that will make your life so much easier. Not only will it solve all the application security concerns we discussed earlier, but it also has a nice feature for you - the option to use Microsoft Entra ID Governance products!

Now, for those lucky folks with Entra ID Governance, this module will not only handle application security, but it will also automate access governance. It creates an Access Package for each application role assignment. This means you can delegate access management to the application owner team without giving them unnecessary permissions. No more red-tapism or service desk routes for them!

I've already documented all the necessary components in the module, so you don't have to worry about repeating them here. But let me give you a quick overview of how this module works. I even added an example to showcase its awesomeness!

With this module, you can effortlessly create an application in your tenant, complete with Application API Permissions and Application Roles. You can assign these roles to access groups, if you have the **Entra ID Governance** license add the groups to the Access Package Catalog, and even create the Access Packages policy to require approval for access package deployment. Phew, that's a lot of power in one module!

So, get ready to embrace the magic of Terraform and Entra ID Governance. It's time to automate, delegate, and conquer the world of application security and access management. And hey, who said coding can't be fun? Let's make security a little less serious and a lot more exciting!

```terraform
data "azuread_application_published_app_ids" "well_known" {}

data "azuread_service_principal" "msgraph" {
  client_id = data.azuread_application_published_app_ids.well_known.result["MicrosoftGraph"]
}

data "azuread_service_principal" "TestAPI" {
  client_id = "6e3df5f1-974f-4305-9361-948f43cc43dd"
}

data "azuread_domains" "example" {
  only_initial = true
}

module "test_application" {
  source = "./modules/EntraIDApplication"

  display_name       = "Test Application"
  identifier_uris    = ["https://test-application"]
  sign_in_audience   = "AzureADMyOrg"
  path_to_logo_image = "GCN3.png"
  app_roles = [
    {
      description  = "Test Application Role"
      display_name = "Test Application"
      value        = "TestApplication Role"
    },
    {
      description  = "Test Application Role2"
      display_name = "Test Application Role 2"
      value        = "TestApplicationRole2"
    }
  ]
  app_role_assignment_required  = true
  description                   = "Test Application Description"
  preferred_single_sign_on_mode = "saml"
  claims_mapping_policy = {
    claims_schema = [
      {
        id              = "id"
        jwt_claim_type  = "name"
        Saml_Claim_Type = "name"
        source          = "user"
      }
    ]
    include_basic_claim_set = "true"
    version                 = "1"
  }

  generate_certificate                               = false
  generate_secret                                    = false
  generate_catalog_access_package                    = true
  approver_group_name                                = "Assigned Group"
  access_package_assignment_policy_approval_required = true
  object_owner_upn                                   = "SuryenduB@03z3s.onmicrosoft.com"
  api_access = [{
    api_client_id = data.azuread_application_published_app_ids.well_known.result["MicrosoftGraph"]
    role_ids = [data.azuread_service_principal.msgraph.app_role_ids["Group.Read.All"],
      data.azuread_service_principal.msgraph.app_role_ids["User.Read.All"],
    ]
    scope_ids = [
      data.azuread_service_principal.msgraph.oauth2_permission_scope_ids["User.ReadWrite"],

    ]


    },
    {
      api_client_id = data.azuread_service_principal.TestAPI.client_id
      role_ids = [
        data.azuread_service_principal.TestAPI.app_role_ids["Files.ReadUser"],
      ]

      scope_ids = [
        data.azuread_service_principal.TestAPI.oauth2_permission_scope_ids["Files.Read"],

      ]
    },
  ]



}


output "access_package" {
  value = module.test_application.access_package
}

output "access_package_url" {
    value = [
      for access_package in module.test_application.access_package : "https://myaccess.microsoft.com/@${data.azuread_domains.example.domains.0.domain_name}#/access-packages/${access_package.id}"
    ]
    
}




output "azuread_application_id" {
  value = module.test_application.azuread_application.application_id
  
}

output "azuread_application_client_id" {
  value = module.test_application.azuread_application.client_id
  
}

output "azuread_application_object_id" {
  value = module.test_application.azuread_application.object_id
  
}
```

## What

In my previous [article](https://suryendub.github.io/2023-11-09-ca-zero-trust-terraform/#use-github-actions-for-automated-ring-base-deployment-of-new-ca-policy), I have already explained how you can use Federated Authentication in the GitHub Actions to deploy Terraform code to Entra ID Tenants. Assign the application necessary graph API permissions as described in the prerequisite sections of the module documentation and you are good to go.

### 1. Application Registration is created in the tenant

![Application Registration](/assets/img/terramod3.png)
![Application Registration Properties](/assets/img/terramod4.png)

### 2. Application API Permissions are added to the Application

![Application API Permissions](/assets/img/terramod5.png)

### 3. Application Roles are added to the Application

![Application Roles](/assets/img/terramod6.png)

### 4. Application Manifest is updated

![Application Manifest](/assets/img/terramod8.png)

### 5. Application Owners are added to the Application

![Application Owners](/assets/img/terramod9.png)

### 6. ID Token and Access Token are not enabled for the Application.

![Application Manifest](/assets/img/terramod7.png)

### 7. Service Principal is created for the Application in the tenant for role assignment and SSO

![Service Principal](/assets/img/terramod10.png)

### 8. Application Roles are assigned to the Access Groups created

![Application Role Assignment](/assets/img/terramod11.png)

### 9. A Claim Mapping Policy is created in the tenant

![Claim Mapping Policy](/assets/img/terramod13.png)

### 10. Claim Mapping Policy is assigned to the Service Principal

![Claim Mapping Policy Assignment](/assets/img/terramod14.png)

### 11. A Catalog is Created for the Application and the Access Groups are added to the Catalog

![Catalog](/assets/img/terramod12.png)
![Catalog Resource](/assets/img/terramod15.png)

### 12. Access Package is created for the Application and the Access Groups are added to the Access Package

![Access Package](/assets/img/terramod16.png)

### 13. Access Package Policy is created for the Access Packages and added to the Access Package

![Access Package Policy](/assets/img/terramod17.png)

![Access Package Policy Approval Stage](/assets/img/terramod18.png)

## Usage

So here is how I envision anyone using this module. You can fork the repository in your GitHub Organization and create a branch protection rule to ensure that no one can merge or push code to the main branch without a review and no one can directly This will ensure that no one can create an application without a review. You can then create a new branch and add the code to create the application in the branch. Once you are done, you can create a pull request and add the necessary reviewers. Once the pull request is approved, you can merge the code to the main branch. This will trigger the GitHub action workflow to deploy the terraform code, and the new application will be created in the tenant. You can then add the necessary application owners to the application and they can manage the application from there on. You need to add the necessary application owners as it is a required parameter for the module. This way, you can ensure that the application is created securely and the application owners can manage the application from there on.

![Flow Diagram for Gitops Flow](/assets/img/terramod2.png)

## What's Next

I am planning to add a few more features to the module. I am planning to add the ability to create access reviews to periodically review the groups. I am also planning to add the ability to create a new approver group and add the group as an approver for the access review. Share any more features that you would like to see in the module. I am not a good coder, if anyone like to contribute and improve the module, please feel free to create a pull request. I will be happy to review and merge the code.

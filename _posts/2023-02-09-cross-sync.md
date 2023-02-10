---
layout: post
title: Cross-Tenant Synchronization | A New Collaboration Experience in Azure Active Directory
subtitle: How to Connect Users Across Multiple Tenants in a Multi-Tenant Organization
cover-img: /assets/img/cross-tenant.jpg
thumbnail-img: /assets/img/cross-tenant.jpg
share-img: /assets/img/cross-tenant.jpg
tags: [ Azure Active Directory, Multi-Tenant Organization, Cross-Tenant Synchronization, Collaboration, External Identities, Microsoft Identity Manager, Azure AD B2B Collaboration]

---




# Cross Tenant Sync

[Cross-tenant synchronization](https://learn.microsoft.com/en-us/azure/active-directory/multi-tenant-organizations/cross-tenant-synchronization-overview) is a new feature in Azure Active Directory for connecting individuals across multiple tenants in a [Multi-tenant organization](https://learn.microsoft.com/en-us/azure/active-directory/multi-tenant-organizations/).

## **Usecase**
 
  To collaborate between businesses, to give access to applications, share point, and teams sites, individuals from one tenant can be invited to another as external identities and access packages can be created through entitlement management. This includes creating access packages and pre-approving certain domains, giving control over the approval process.

Now consider these scenarios 

1. Two organizations merged, for example, Contoso acquired Fabrikam. Contoso is currently trusted as the IDP of required SAAS applications of the organization.

2. If your organization is restructured into two legally independent business units, for example, organization Contoso is restructured into two separate entities Contoso and Fabrikam.

In this scenario, the aim is to automatically add a large number of individuals from the Fabrikam tenant as guests in the Contoso tenant for smooth collaboration within the same organization, taking into account privacy concerns.



## Existing Solutions and Drawbacks
1. **Microsoft Identity Manager (MIM)**: We can use the [MIM graph connector](https://learn.microsoft.com/en-us/microsoft-identity-manager/microsoft-identity-manager-2016-connector-graph) to provision users between two Azure AD Tenants as guest users, to automate lifecycle management , update attribute of guest Azure AD Users.  However, MIM is a legacy solution, and managing MIM Infrastructure is nightmarish. To implement the scenario described above you need to be highly skilled individuals with extensive knowledge of various MIM components.

2. **Azure AD B2B Collaboration** : Organizations can continue to use B2B Collaboration, however, guest user lifecycle management is not automated. If a user is removed from the source tenant, it will continue to exist in the target tenant. 


With new **Azure AD Cross Tenant synchronization** you can achieve following:

*	Create B2B users in your target tenant
*	Remove B2B users in your target tenant
*	Keep user attributes synchronized between your source and target tenants
*  Minimise user friction by allow admins to consent to sharing data across tenants.

![cross-collab](/assets/img/cross-tenant.jpg)

Follow the following steps to configure **Cross Tenant Synchronization**:

### 1.**Enable cross-tenant synchronization and auto-redemption in the target tenant**

* Sign in to the Azure portal as an administrator in the target tenant.

* Select Azure Active Directory > External Identities.

* Select Cross-tenant access settings.

* On the Organization settings tab, select Add organization.

* Add the source tenant by typing the tenant ID or domain name and selecting Add.


* Under Inbound access of the added organization, select Inherited from default.
![Add from default policy](/assets/img/cross-tenant-external.jpg)
* Select the **Trust Settings** tab, select **Customized settings** ,  notice the **Consent Prompt** option, here is where you will want to enable this feature as it will suppress consent prompts from apps in your target tenant for synced users from synced tenant.  

    ![Consent prompt](/assets/img/cross-tenant-external-2.jpg)

* Select the Cross-tenant sync (Preview) tab.

* Check the Allow users sync into this tenant check box.  
    ![Allow user sync](/assets/img/cross-tenant-external-3.jpg)


* Select Save. 

### 2. **Automatically redeem invitations in the source tenant**



* Sign in to the Azure portal as an administrator of the source tenant.

* Select Azure Active Directory > External Identities.

* Select Cross-tenant access settings.

* On the Organization settings tab, select Add organization.

* Add the target tenant by typing the tenant ID or domain name and selecting Add.

    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-source.jpg)

* Under Outbound access for the target organization, select Inherited from default.
    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-source-2.jpg)

* Select the Trust settings tab.

* Check the Suppress consent prompts for users from my tenant when they access apps and resources in the other tenant check box.

  ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-source-3.jpg)

* Select Save.

### 3. **Create a configuration in the source tenant**

* In the source tenant, select Azure Active Directory > Cross-tenant synchronization (Preview).

    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-crt.jpg)
Select Configurations.

* At the top of the page, select New configuration.

* Provide a name for the configuration and select Create.  
    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-crt-2.jpg)
* It can take up to 15 seconds for the configuration that you just created to appear in the list.

* Select Refresh to retrieve the latest list of configurations.

* In the source tenant, in the configuration list, select your configuration.



* Select Get started.

* Set the Provisioning Mode to Automatic.

* Under the Admin Credentials section, change the Authentication Method to Cross Tenant Synchronization Policy.


* In the Tenant Id box, enter the tenant ID of the target tenant.

* Select Test Connection to test the connection.
    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-crt-3.jpg)




    ![Screenshot that shows the Add organization pane to add the source tenant.](/assets/img/cross-tenant-crt-4.jpg)

* Select Save.

* Mappings and Settings sections appear.

* In this section, you can specify an email address for receiving sync error notifications, activate the feature to prevent accidental deletion, and select either to synchronize all users and groups (not recommended) or to limit it to the designated users and groups (recommended). Go ahead and choose your preferred option. During testing, it's best to leave this on the default setting first.

    ![prov](/assets/img/crt-prov-setting.jpg)

* Close the Provisioning page.

### 4.**Define users to synchronize (Based on group membership and attribute value).** 
* Navigate to the Users and Groups section on the left side. From this location, you can select the users or groups to be included in the synchronization.  
    ![prov](/assets/img/crt-prov-user.jpg)

* If you need to further limit the scope of the synchronization, you can use scoping filters. To do this, go back to the Provisioning blade and select "Provision Azure Active Directory Users" under the Mappings subsection.
    
    ![prov](/assets/img/cross-tenant-mapping.jpg)
* From this location, select "All records" under the Source Object Scope. This enables you to determine the scope of user provisioning based on user attributes. For instance, you could limit it to only target users within the Marketing department.  
    ![prov](/assets/img/cross-tenant-scoping-filter.jpg)


### 5. **What attributes to synchronize (name, department, directory extensions, etc.)** 
      
* Any desired transformations, such as adding the domain to the end of the display name.

    ![prov](/assets/img/cross-tenant-prov-attribute-mapping.jpg)


### 6. **Test provision on demand**
* Navigate to the **Provision on demand** section on the left side. From this location, you can select the user.  
    ![prov](/assets/img/cross-tenant-ondemand-prov.jpg)
* Click on provision, on successful provision you will see something similar to the screenshot below.  
    ![success](/assets/img/cross-tenant-ondemand-prov-2.jpg)
* Verify the same in the target tenant.
    ![prov](/assets/img/cross-tenant-ondemand-prov-3.jpg)  
 
 >In conclusion, the Azure Active Directory's new Cross-Tenant Synchronization feature is a game-changer for organizations looking to collaborate and share resources across multiple tenants. It provides a streamlined solution to address the challenges posed by existing solutions such as Microsoft Identity Manager, which requires extensive technical knowledge and resources, and Azure AD B2B Collaboration, which lacks automated guest user lifecycle management. With Cross-Tenant Synchronization, organizations can easily add, remove and keep user attributes synchronized between source and target tenants, while ensuring that privacy concerns are addressed through the consent prompt feature. The three-step configuration process is straightforward and can be accomplished by administrators within the organization. This new feature aligns with the philosophy of engineers to make things simple, efficient and effective, providing a solution that reduces the barriers to collaboration, making it easier for businesses to work together.

  





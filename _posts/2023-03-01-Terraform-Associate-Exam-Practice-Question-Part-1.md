---
layout: post
title: Terraform Associate Exam Practice Questions - Part 1
subtitle: Terraform Associate Exam Practice Questions
cover-img: /assets/img/TerraformAssociate.png
thumbnail-img: /assets/img/TerraformAssociate.png
share-img: /assets/img/TerraformAssociate.png
tags: [IAC,Terraform,Exam,Certification]
---


# Terraform Associate Level Questions Part 1


> What is idempotent in IAC?

  In the context of Infrastructure as Code (IaC), idempotence refers to the property of a configuration management tool or script that ensures that multiple executions of the same script or tool result in the same outcome as the first execution, regardless of the number of times the script or tool is run.In other words, an idempotent IaC tool or script will only make changes to the infrastructure that are necessary to achieve the desired state, and will not make redundant changes or cause unintended side effects.This is an important concept in IaC because it allows for consistent and predictable infrastructure deployment and management, reduces the risk of errors and misconfigurations, and simplifies troubleshooting and debugging.


> What is cloud-agnostic in terms of provisioning tools?**


 Cloud-agnostic provisioning tools are tools that allow for the deployment and management of infrastructure resources across multiple cloud environments or providers, without being tied to a specific cloud platform or vendor.These tools are designed to provide a high degree of flexibility and portability, allowing organizations to leverage multiple cloud providers or switch between providers as needed, without having to re-write their infrastructure code or modify their deployment processes.
Cloud-agnostic provisioning tools typically provide a layer of abstraction between the infrastructure code and the underlying cloud resources, allowing for the use of a common language or syntax to describe the infrastructure, regardless of the cloud platform being used. This makes it easier to manage and maintain infrastructure resources across multiple clouds, and helps to avoid vendor lock-in.Examples of cloud-agnostic provisioning tools include Terraform, Ansible, and Chef, which support multiple cloud platforms and can be used to manage infrastructure resources in a consistent and portable manner.

> What is the Terraform State?

 In Terraform, state refers to the current state of the infrastructure resources being managed by Terraform. The Terraform state is a representation of the resources that Terraform is managing, including their current configuration, metadata, and relationships with other resources.The Terraform state is stored in a file called the state file, which is typically named terraform.tfstate. The state file is JSON formatted and contains information about all of the resources managed by Terraform, including their current state, attributes, and dependencies.The state file is used by Terraform to determine the changes that need to be made to the infrastructure resources in order to bring them into the desired state. When Terraform runs, it reads the state file to determine the current state of the resources, and then compares that to the desired state as defined in the Terraform configuration files. Based on this comparison, Terraform determines the actions that need to be taken to bring the resources into the desired state.It's important to manage the Terraform state file carefully, as it contains sensitive information about the infrastructure being managed, and can be used to modify or destroy resources if it falls into the wrong hands. Terraform provides several mechanisms for managing state, including storing state remotely using services like Terraform Cloud or AWS S3, or using state locking to prevent multiple users or processes from modifying the state file simultaneously.

> What are the Providers? 

 In Terraform, a provider is a plugin that enables Terraform to interact with a specific cloud, service, or technology platform. Providers are used to define the resources that Terraform can create, update, and manage within a given cloud or platform.Each provider is responsible for implementing the necessary API interactions, authentication mechanisms, and resource definitions required to manage resources within a particular cloud or platform. Providers can be developed and maintained by the platform vendors themselves or by third-party developers, and Terraform maintains a registry of providers that are available for use.When a Terraform configuration file references a particular resource, such as an EC2 instance in AWS or a virtual machine in Azure, Terraform uses the appropriate provider plugin to create, update, or delete that resource as needed. Terraform providers are typically defined in the configuration files using a provider block, which specifies the name of the provider and any required configuration parameters, such as access keys or API endpoints.Using providers in Terraform allows infrastructure to be defined in a cloud-agnostic way, making it possible to manage resources across multiple cloud providers or technology platforms using a common configuration language and toolset. Some of the most commonly used Terraform providers include AWS, Azure, Google Cloud Platform, and Kubernetes.
```terraform
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "example-instance"
  }
}

```
In this example, the **provider** block specifies that the **AWS** provider should be used, and sets the region to **us-west-2**.The resource block defines an EC2 instance resource, using the aws_instance resource type provided by the AWS provider. The block specifies the AMI ID and instance type for the instance, as well as a name tag to identify the instance.When this Terraform configuration file is applied, Terraform will use the AWS provider to create an EC2 instance in the us-west-2 region, using the specified AMI ID and instance type, and apply the name tag to the instance. If the instance already exists, Terraform will update it to match the desired state specified in the configuration file.

> How do we define multiple Provider configurations?

 To define multiple provider configurations in Terraform, you can simply add multiple provider blocks to your configuration file, each with a unique alias to distinguish between them. Here's an example of how you could define multiple Azure provider configurations in Terraform:
```terraform
provider "azurerm" {
  features {}
}

provider "azurerm" {
  alias = "secondary"
  features {}
}

resource "azurerm_virtual_network" "primary" {
  name                = "primary-network"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "primary-resource-group"
}

resource "azurerm_virtual_network" "secondary" {
  provider            = azurerm.secondary
  name                = "secondary-network"
  address_space       = ["192.168.0.0/16"]
  location            = "eastus"
  resource_group_name = "secondary-resource-group"
}

```
 this example, there are two provider blocks that define two separate Azure provider configurations, one named ***azurerm*** and the other named ***azurerm.secondary***. The features block is empty, indicating that the default feature set for each provider should be used. Two virtual networks are then defined as ***azurerm_virtual_network*** resources. The first virtual network, named primary, uses the default azurerm provider configuration, and is created in the westus region with a resource group named primary-resource-group. The second virtual network, named secondary, uses the azurerm.secondary provider configuration, which is aliased as secondary, and is created in the eastus region with a resource group named secondary-resource-group.By using different provider configurations, you can deploy and manage resources across multiple Azure environments or subscriptions using a single Terraform configuration file.

6.**What is the CLI configuration File?**

 The CLI configuration file in Terraform, commonly known as ***terraform.rc*** is a file that allows you to configure various settings related to the Terraform CLI, such as default values for command-line options, logging settings, and plugin directory locations. The CLI configuration file is optional and can be created in the user's home directory or in the directory where Terraform is being executed.

Here's an example of how you can use the CLI configuration file to set a default value for the ***plan*** command's out option:

```makefile
# terraform.rc file
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"

default_plan_out = "terraform.tfplan"

```
In this example, the ***default_plan_out*** option is used to set a default value for the plan command's -out option, which specifies the path to the file where the Terraform execution plan should be saved. With this configuration, whenever the ***plan*** command is executed without specifying the ***-out*** option, Terraform will use the value specified in the ***terraform.rc*** file. Other settings that can be configured in the CLI configuration file include the location of the plugin cache directory, the logging level for Terraform output, and the settings for the Terraform backend. By configuring the CLI using the ***terraform.rc*** file, you can customize the behavior of the Terraform CLI to better fit your workflow and environment.

7. **How do you inspect the current state of the infrastructure applied?**  
 You can inspect the current state of the infrastructure managed by Terraform using the ***terraform state*** command. This command allows you to view and manage the state of resources that are being managed by Terraform.

Here are some of the most common subcommands that can be used with ***terraform state***:

* **list:** Lists all resources that are currently being managed by Terraform.
* **show:** Displays detailed information about a specific resource or all resources.
* **mv:** Moves a resource to a new name or location in the state file.
* **rm:** Removes a resource from the state file.
For example, to view the current state of all resources managed by Terraform, you can run:

```terraform
terraform state list

```
To view detailed information about a specific resource, such as an EC2 instance, you can run:
```
terraform state show aws_instance.example

```
In this example, ***aws_instance.example*** is the resource name as defined in the Terraform configuration file.

You can also use the -json option to output the state information in JSON format, which can be useful for parsing and processing the state data programmatically.

By inspecting the current state of the infrastructure managed by Terraform, you can ensure that the desired state matches the actual state, and identify any discrepancies or drifts that may have occurred due to changes made outside of Terraform.

By default, provisioners that fail will also cause the Terraform apply itself to fail. Is this true?

True



8. **By default, provisioners that fail will also cause the Terraform apply itself to fail. How do you change this?**

The on_failure setting can be used to change this.

The allowed values are:

continue: Ignore the error and continue with creation or destruction.

fail: Raise an error and stop applying (the default behavior). If this is a creation provisioner, taint the resource.
```terraform
// Example

resource "aws_instance" "web" {

  # ...

  provisioner "local-exec" {

    command  = "echo The server's IP address is ${self.private_ip}"

    on_failure = "continue"

  }

}

```

9. **How do you define destroy provisioner and give an example?**

You can define destroy provisioner with the parameter when
```terraform
provisioner "remote-exec" {

    when = "destroy"

    # <...snip...>

}

```

10. **Your application team resources want to contribute to IaC coding practices . They want you to refactor your code to be reusable , and wants you to create as many modules as possible , along with 3-4 levels of child modules , and also modules as wrappers for even simple resource types. What should your response be to the above?**

You should reject their request , and tell them that rather module composition should be used . Module nesting should not be more than 1 level , and also , modules should not be simple wrappers around basic resources.


11. **Your company has been using Terraform Cloud for a some time now . But every team is creating their own modules , and there is no standardization of the modules , with each team creating the resources in their own unique way . You want to enforce a standardization of the modules across the enterprise . What should be your approach.**

Implement a Private module registry in Terraform cloud , and ask teams to reference them.

12. 
```terraform
resource "aws_s3_bucket" "example" {
  bucket = "my-test-s3-terraform-bucket"
  …}

resource "aws_iam_role" "test_role" {
  name = "test_role"
  …}

  ```
**Due to the way that the application code is written , the s3 bucket must be created before the test role is created , otherwise there will be a problem. How can you ensure that?**

Add explicit dependency using depends_on . This will ensure the correct order of resource creation.

13. **You have created a terraform script that uses a lot of new constructs that have been introduced in terraform v0.12  . However , many developers who are cloning the script from your git repo , are using v0.11 , and getting errors , and raising hundreds of tickets. What can be done from your end to solve this problem?**

Use the terraform setting ***‘required_version’*** and set it to > 0.12. This will ensure that developers are forced to use v0.12 , and does not use anything else.

14. **Your developers are facing a lot of problem while writing complex expressions involving difficult interpolations . They have to run the terraform plan every time and check whether there are errors , and also check terraform apply to print the value as a temporary output for debugging purposes. What should be done to avoid this?**

Use terraform console command to have an interactive UI with full access to the underlying terraform state to run your interpolations , and debug at real-time.

15.**Please identify the offerings which are unique to Terraform Enterprise , and not available in either Terraform OSS , or Terraform Cloud. There can be more than one answer.**

A. **SAML/SSO**  
B. Sentinel  
C. **Audit Logs**  
D. **Private Network Connectivity**  
E. Full API Coverage  
F. VCS Integration  
G. **Clustering**

16. **Your team has started using terraform OSS in a big way , and now wants to deploy multi region deployments (DR) in aws using the same terraform files . You want to deploy the same infra (VPC,EC2 …) in both us-east-1 ,and us-west-2 using the same script , and then peer the VPCs across both the regions to enable DR traffic. But , when you run your script , all resources are getting created in only the default provider region. What should you do? Your provider setting is as below-**
```terraform
# The default provider configuration
provider "aws" {
  region = "us-east-1"
}
```
Use provider alias functionality , and add another provider for us-west region . While creating the resources using the tf script , reference the appropriate provider (using the alias).

17. **You have 5-7 developers working on your terraform project (using terraform OSS), and you saved the terraform state in a remote S3 bucket . However , sometimes you are observing inconsistencies in the provisioned infrastructure / failure in the code . You have traced this problem to simultaneous/concurrent runs of terraform apply command for 2/more developers . What can you do to fix this problem ?**

Enable terraform state locking for the S3 backend using DynamoDB table. This prevents others from acquiring the lock and potentially corrupting your state.

18. **You have written a terraform IaC script which was working till yesterday , but is giving some vague error from today , which you are unable to understand . You want more detailed logs that could potentially help you troubleshoot the issue , and understand the root cause. What can you do to enable this setting? Please note , you are using terraform OSS.**  
Enable TF_LOG to the log level DEBUG , and then set TF_LOG_PATH to the log sink file location . Terraform debug logs will be dumped to the sink path ,even in terraform OSS

19. **What does terraform refresh command do ?**  
Terraform refresh syncs the state file with the real world infrastructure.  It does not affect the actual target infra / the terraform code file in any fashion.
20. **Given the below resource configuration -**
```terraform
resource "aws_instance" "web" {
  # ...
  count = 4
}
```
21. **What does the terraform resource address ‘aws_instance.web’ refer to?** 

It refers to all 4 web instances , together , for further individual segregation , indexing is required , with a 0 based index.

---
layout: post
title: Terraform Associate Exam Practice Questions - Part 2
subtitle: Terraform Associate Exam Practice Questions
cover-img: /assets/img/TerraformAssociate.png
thumbnail-img: /assets/img/TerraformAssociate.png
share-img: /assets/img/TerraformAssociate.png
tags: [IAC,Terraform,Exam,Certification]
---


# Terraform Associate Level Questions Part 2

> How many ways you can assign variables in the configuration?
* Define a variable within a module or configuration file
    ```terraform
    variable "region" {
    type = string
    }
    ```

* Assign a value to the variable through the command line
    ```
    terraform apply -var="region=us-west-2"
    ```
* Assign a value to the variable through an environment variable
export ***TF_VAR_region=us-west-2***

* Load variable values from a tfvars file
    ```
    terraform apply -var-file=terraform.tfvars
    ```
* UI input
   
>Does environment variables support List and map types?   

 No. Environment variables can only populate string-type variables. List and map type variables must be populated via one of the other mechanisms.

> How do you view outputs and queries them?  

You can query the output with the following command
```
terraform output ip
```
> What is Sentinel ?

***Sentinel*** is a policy-as-code framework developed by HashiCorp, the same company that created Terraform. Sentinel provides a way to define and enforce policies on infrastructure code, ensuring that it complies with organizational and regulatory requirements.

Sentinel policies are written in a high-level, human-readable language that is designed to be accessible to both developers and non-developers. Policies can be defined to enforce a wide range of requirements, such as:

- Resource naming conventions
- Security and compliance standards
- Resource usage limits
- Cost management

Sentinel policies are evaluated during the Terraform plan and apply phases, allowing developers to catch and fix policy violations early in the development process.

Sentinel can be integrated with various tools in the HashiCorp ecosystem, such as Terraform Cloud, Vault, and Consul. It can also be integrated with third-party tools through its API.

Overall, Sentinel provides a powerful tool for organizations to ensure that their infrastructure code is compliant with their policies and standards, reducing the risk of security breaches, compliance violations, and other issues.  
>What is the flag you should use to upgrade modules and plugins a part of their respective installation steps?

 ```
terraform init -upgrade

 ```

>  When you are doing initialization with terraform init, you want to skip plugin installation. What should you do?  

When running `terraform init`, Terraform will automatically download and install any required plugins for the specified provider. However, in some cases, you may want to skip plugin installation. One example might be when you are working offline and cannot access the necessary plugin repositories.

To skip plugin installation during `terraform init`, you can use the `-plugin-dir` flag with a directory path that already contains the required plugin binaries. Here is an example:
```terraform
terraform init -plugin-dir=/path/to/plugins
```

This will cause Terraform to skip plugin installation and use the specified plugin directory instead. Note that this directory should contain all of the required plugin binaries for the specified provider, and the binaries should be compatible with the version of Terraform that you are using.

In general, it is recommended to let Terraform handle plugin installation automatically. However, if you need to skip plugin installation for some reason, using the `-plugin-dir` flag is a way to achieve this.
```terraform
terraform init -get-plugins=false
```
Skips plugin installation. Terraform will use plugins installed in the user plugins directory, and any plugins already installed for the current working directory. If the installed plugins aren't sufficient for the configuration, init fails.

> What are implicit and explicit dependencies?


In Terraform, dependencies are relationships between resources that affect the order in which they are created and destroyed. There are two types of dependencies in Terraform: implicit and explicit.

#### Implicit Dependencies:

Implicit dependencies are relationships between resources that are automatically detected by Terraform based on the resource configuration. For example, if you create an Azure resource group and then create an Azure virtual machine that uses that resource group, Terraform will automatically detect the dependency between the two resources.

Here is an example of an implicit dependency in Terraform using the AzureRM provider:
```terraform
resource "azurerm_resource_group" "example" {
name = "example-rg"
location = "westus2"
}

resource "azurerm_virtual_machine" "example" {
name = "example-vm"
location = "westus2"
resource_group_name = azurerm_resource_group.example.name
network_interface_ids = [azurerm_network_interface.example.id]
}

resource "azurerm_network_interface" "example" {
name = "example-nic"
location = "westus2"
resource_group_name = azurerm_resource_group.example.name
}
```

In this example, the `azurerm_virtual_machine` resource has an implicit dependency on the `azurerm_resource_group` and `azurerm_network_interface` resources because it references them in its configuration.

#### Explicit Dependencies:

Explicit dependencies are relationships between resources that are defined explicitly in the Terraform configuration. You can use the `depends_on` attribute to specify explicit dependencies between resources.

Here is an example of an explicit dependency in Terraform using the AzureRM provider:
```terraform
resource "azurerm_resource_group" "example" {
name = "example-rg"
location = "westus2"
}

resource "azurerm_storage_account" "example" {
name = "examplestorageaccount"
resource_group_name = azurerm_resource_group.example.name
location = azurerm_resource_group.example.location
account_tier = "Standard"
account_replication_type = "LRS"
}

resource "azurerm_storage_blob" "example" {
name = "exampleblob"
storage_account_name = azurerm_storage_account.example.name
storage_container_name = "examplecontainer"
type = "Block"
depends_on = [azurerm_storage_account.example]
}
```

In this example, the `azurerm_storage_blob` resource has an explicit dependency on the `azurerm_storage_account` resource because it specifies the `depends_on` attribute with the `azurerm_storage_account.example` reference.

> How do you save the execution plan?
```
terraform plan -out=tfplan

# you can use that file with apply

terraform apply tfplan
```
>What is a partial configuration in terms of configuring Backends?

In Terraform, a partial configuration is a way to specify only a subset of the full configuration for a particular resource or backend. 
```terraform
terraform {
  backend "azurerm" {
    storage_account_name = "mytfstatestorage"
    container_name       = "tfstate"
   
  }
}

```
> What is the command fmt?

The `terraform fmt` command is used to format Terraform configuration files to match the standard formatting rules. This command updates the formatting of your configuration files in place and makes them more readable and consistent.

Here are the different options available with `terraform fmt`:

- `-list=false`: Disables listing of files whose formatting differs from the standard.
- `-write=false`: Disables in-place file updates, instead outputs the result to stdout.
- `-diff`: Outputs the differences between the standard formatting and the file's formatting.
- `-check`: Returns a non-zero exit code if the formatting differs from the standard, useful in a CI/CD pipeline.

Here is an example of using `terraform fmt` with options:

```bash
terraform fmt -diff -check -list=true
```

In this example, `terraform fmt` is run with the `-diff` option to show the differences between the standard formatting and the current file formatting, the `-check` option to return a non-zero exit code if the formatting differs from the standard, and the `-list=true` option to list all the files that differ from the standard formatting.

Remember that running `terraform fmt` will modify your files in place, so be sure to use version control to track any changes made to your files.

> What is the command taint?   

The `terraform taint` command is used to mark a resource in the Terraform state as "tainted", meaning that it needs to be recreated on the next `terraform apply` command. This can be useful in situations where the state of a resource becomes corrupted or out-of-sync with the actual infrastructure.

To use `terraform taint`, you must first identify the resource that needs to be marked as tainted. You can find the resource address in the Terraform state file, or by using the `terraform state list` command. Once you have the resource address, you can use the `terraform taint` command to mark it as tainted.

Here is an example of using `terraform taint` with the Azure provider:

```bash
terraform taint azurerm_virtual_machine.example
```

In this example, the `terraform taint` command is used to mark the `azurerm_virtual_machine.example` resource as tainted. This means that the next time `terraform apply` is run, the virtual machine will be destroyed and recreated.

There are a few options available with `terraform taint`:

- `-lock`: Locks the state file before making changes to it.
- `-state=foo`: Specifies the path to the Terraform state file.
- `-backup=path`: Specifies the path to backup the state file before modifying it.
- `-config=path`: Specifies the path to the Terraform configuration file directory.

It's important to use caution when using `terraform taint`, as it can result in data loss and other unintended consequences. It should only be used in situations where it is necessary to recreate a resource from scratch.

> what is crash.log in terraform?

In Terraform, a crash.log file is created when Terraform encounters an error or unexpected behavior during an operation. This log file contains detailed information about the error, including the time of the error, the affected resource(s), and the stack trace.The crash.log file can be useful for troubleshooting and debugging purposes, as it provides a detailed record of what happened during the operation that caused the error. Additionally, the log file can be used to share information about the error with Terraform's support team or community, in order to get help resolving the issue. By default, the crash.log file is stored in the .terraform directory of the Terraform working directory.








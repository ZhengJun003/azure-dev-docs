---
title: Quickstart - Install and configure Terraform to provision Azure resources 
description: Learn how to install and configure Terraform to create Azure resources.
keywords: azure devops terraform install configure
ms.topic: quickstart
ms.date: 04/26/2020
---

# Quickstart: Install and configure Terraform to provision Azure resources
 
Terraform provides an easy way to define, preview, and deploy cloud infrastructure by using a [simple templating language](https://www.terraform.io/docs/configuration/syntax.html). This article describes the necessary steps to use Terraform to provision resources in Azure.

[!INCLUDE [hashicorp-support.md](includes/hashicorp-support.md)]

## Prerequisites

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]

## Install Terraform

By default, the latest version of Terraform is installed for use in the [Azure Cloud Shell](/azure/cloud-shell/overview). If you choose to install Terraform locally, complete this step; otherwise, continue to [Configure Terraform access to Azure](#configure-terraform-access-to-azure).

1. [Install Terraform](https://www.terraform.io/downloads.html) specifying the appropriate package for your operating system.
1. The download contains a single executable file. Define a global path to the executable based on your operating system:
    - [Linux or MacOS](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux)
    - [Windows](https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows).
1. Verify the global path configuration with the `terraform` command. If Terraform is found and runs, a list of available Terraform options displays:

    ```console
    azureuser@Azure:~$ terraform
    Usage: terraform [--version] [--help] <command> [args]
    ```
    
## Configure Terraform access to Azure

To enable Terraform to provision resources into Azure, create an [Azure AD service principal](/cli/azure/create-an-azure-service-principal-azure-cli). The service principal grants your Terraform scripts to provision resources in your Azure subscription.

If you have multiple Azure subscriptions, first query your account with [az account list](/cli/azure/account#az-account-list) to get a list of subscription ID and tenant ID values:

```azurecli-interactive
az account list --query "[].{name:name, subscriptionId:id, tenantId:tenantId}"
```

To use a selected subscription, set the subscription for this session with [az account set](/cli/azure/account#az-account-set). Set the `SUBSCRIPTION_ID` environment variable to hold the value of the returned `id` field from the subscription you want to use:

```azurecli-interactive
az account set --subscription="${SUBSCRIPTION_ID}"
```

Now you can create a service principal for use with Terraform. Use [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac), and set the *scope* to your subscription as follows:

```azurecli-interactive
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"
```

Your `appId`, `password`, `sp_name`, and `tenant` are returned. Make a note of the `appId` and `password`.

## Configure Terraform environment variables

To configure Terraform to use your Azure AD service principal, set the following environment variables, which are then used by the [Azure Terraform modules](https://registry.terraform.io/modules/Azure). You can also set the environment if working with an Azure cloud other than Azure public.

- `ARM_SUBSCRIPTION_ID`
- `ARM_CLIENT_ID`
- `ARM_CLIENT_SECRET`
- `ARM_TENANT_ID`
- `ARM_ENVIRONMENT`

You can use the following sample shell script to set those variables:

```bash
#!/bin/sh
echo "Setting environment variables for Terraform"
export ARM_SUBSCRIPTION_ID=your_subscription_id
export ARM_CLIENT_ID=your_appId
export ARM_CLIENT_SECRET=your_password
export ARM_TENANT_ID=your_tenant_id

# Not needed for public, required for usgovernment, german, china
export ARM_ENVIRONMENT=public
```

## Run a sample script

Create a file `test.tf` in an empty directory and paste in the following script.

```hcl
provider "azurerm" {
  # The "feature" block is required for AzureRM provider 2.x. 
  # If you are using version 1.x, the "features" block is not allowed.
  version = "~>2.0"
  features {}
}
resource "azurerm_resource_group" "rg" {
        name = "testResourceGroup"
        location = "westus"
}
```

Save the file and then initialize the Terraform deployment. This step downloads the Azure modules required to create an Azure resource group.

```bash
terraform init
```

The output is similar to the following example:

```console
* provider.azurerm: version = "~> 0.3"

Terraform has been successfully initialized!
```

You can preview the actions to be completed by the Terraform script with `terraform plan`. When ready to create the resource group, apply your Terraform plan as follows:

```bash
terraform apply
```

The output is similar to the following example:

```console
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.rg
      id:       <computed>
      location: "westus"
      name:     "testResourceGroup"
      tags.%:   <computed>

azurerm_resource_group.rg: Creating...
  location: "" => "westus"
  name:     "" => "testResourceGroup"
  tags.%:   "" => "<computed>"
azurerm_resource_group.rg: Creation complete after 1s
```

## Next steps

In this article, you installed Terraform or used the Cloud Shell to configure Azure credentials and start creating resources in your Azure subscription. To create a more complete Terraform deployment in Azure, see the following article:

> [!div class="nextstepaction"]
> [Create an Azure VM with Terraform](create-linux-virtual-machine-with-infrastructure.md)
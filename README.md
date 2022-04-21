# azure-terrafy-walkthrough
Welcome to the Azure Terrafy guide for importing your existing Azure infrastructure under Terraform management. The installation steps in this guide focus on a Windows deployment, but the import steps are consistent across all environments (MacOS,Unix,Linux,BSD)


# Deployment Steps
 
 ## Dependencies 
 - Install Terraform https://www.terraform.io/downloads
 - Install Azure Terrafy https://github.com/Azure/aztfy
 - PowerShell 7 https://github.com/PowerShell/PowerShell
 - Azure CLI https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-powershell
 - VSCode https://code.visualstudio.com/
 - Windows Terminal https://github.com/microsoft/terminal

 1. Install and download prerequisites
2. Configure VS Code (Extensions)
3. Configure Terraform \ Azure Terrafy  (Extract Terraform executable to "c:\terraform", extract Azure Terrafy to "c:\aztfy")
   - To add the Terraform\ Azure Terrafy executable directory's to your PATH variable:

     - Click on the Start menu and search for Settings. Open the Settings app.
     - Select the System icon, then on the left menu, select the About tab. Under Related settings on the right, select Advanced system settings.
     - On the System Properties window, select Environment Variables.
     - Select the PATH variable, then click Edit.
     - Click the New button, then type in the path where the Terraform & Terrafy executable is located.

# The following steps are completed from the CLI (Windows Terminal) 

## Authenticate to Azure
We need to authenticate to Azure in order for Terrafy to read our target subscriptions \ resource groups.    

### Azure Subscription Configuration:
#### Azure CLI
```
1. az login (login)
2. set azure subscription reference az account set --subscription "my sub"
```

#### Azure PowerShell
```
1. Connect-AzAccount
2. Set-AzContext -Subscription "Subscription String"
```

## Azure Terrafy
Create a new directory and use the tool to generate the supporting Terraform code to recreate all of those resources:
example:
```
 - mkdir aztfy_netrunner_demo
 - cd aztfy_netrunner_demo (This selects our newly created Azure Terrafy working directory)
 ```
 
 ### Terraform Demo Plan Config Example:

See below an example terraform state list that was outputted from the demo terraform configuration files included in this repo. We will use the below state list to verify our imported Azure configuration into Terraform state using Azure Terrafy.
```
Run "terraform state list" in your working directory after a successful "Terraform apply" to your Azure environment. This will output a similar resource list below for cross-reference.
```
```
 azurerm_network_interface.vm_nic
 azurerm_network_security_group.vm_subnet_nsg
 azurerm_resource_group.vm_resource_group
 azurerm_subnet.vm_subnet
 azurerm_subnet_network_security_group_association.vm_subnet_nsg_association
 azurerm_virtual_network.vm_vnet
 azurerm_windows_virtual_machine.vm_01
```
 
 ### Lets Run Azure Terrafy:
 In our working directory run the following command:
 ```   
 - aztfy "your Azure external resource group name"
 ```
 ![Azure Terrafy](/assets/img/image1.png "Terrafy Initialize Screenshot")   
### Accept the defaults, in this example  which included all of the resources.
![Azure Terrafy](/assets/img/image2.png "Terrafy Import List Screenshot")   

### The import process will begin as depicted here:
![Azure Terrafy](/assets/img/image3.png "Terrafy Import Screenshot")   

### Once the process is complete you will be greeted with a similar message's below:
![Azure Terrafy](/assets/img/image4.png "Terrafy End Of Process Screenshot")   
 ```
 Azure Terrafy
  Terraform state and the config are generated at: C:\Users\KevinEvans\win-local-dev\aztfy_netrunner_demo
  ```
  ### Imported Terraform working directory configuration:
  ```    
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          18/04/2022    18:35                .terraform
-a---          18/04/2022    18:39           1071 .aztfyResourceMapping.json
-a---          18/04/2022    18:35           1108 .terraform.lock.hcl
-a---          18/04/2022    18:39           1983 main.tf
-a---          18/04/2022    18:35            181 provider.tf
-a---          18/04/2022    18:39          10208 terraform.tfstate
-a---          18/04/2022    18:39           9291 terraform.tfstate.backup
``` 

### The provider.tf file contains the Terraform block and provider block:
``` 
terraform {
  backend "local" {}
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "3.0.2"
    }
  }
}

provider "azurerm" {
  features {}
}
``` 

### The main.tf file contains definitions for 7 different resources which make up the demo VM deployment:
```
resource "azurerm_network_security_rule" "res-3" {
  access                      = "Allow"
  destination_address_prefix  = "*"
  destination_port_range      = "*"
  direction                   = "Inbound"
  name                        = "rdp"
  network_security_group_name = "acceptanceTestSecurityGroup1"
  priority                    = 100
  protocol                    = "Tcp"
  resource_group_name         = "vm-resources"
  source_address_prefix       = "*"
  source_port_range           = "*"
  depends_on = [
    azurerm_network_security_group.res-0,
  ]
}
resource "azurerm_virtual_network" "res-4" {
  address_space       = ["10.0.0.0/16"]
  location            = "westeurope"
  name                = "iaas-network"
  resource_group_name = "vm-resources"
  depends_on = [
    azurerm_network_security_group.res-0,
  ]
}
resource "azurerm_subnet" "res-5" {
  name                 = "internal"
  resource_group_name  = "vm-resources"
  virtual_network_name = "iaas-network"
  depends_on = [
    azurerm_virtual_network.res-4,
    azurerm_network_security_group.res-0,
  ]
}
resource "azurerm_resource_group" "res-6" {
  location = "westeurope"
  name     = "vm-resources"
}
resource "azurerm_network_security_group" "res-0" {
  location            = "westeurope"
  name                = "acceptanceTestSecurityGroup1"
  resource_group_name = "vm-resources"
  tags = {
    environment = "Production"
  }
  depends_on = [
    azurerm_resource_group.res-6,
  ]
}
resource "azurerm_windows_virtual_machine" "res-1" {
  admin_password        = null # sensitive
  admin_username        = "adminuser"
  custom_data           = null # sensitive
  location              = "westeurope"
  name                  = "vm-01"
  network_interface_ids = ["/subscriptions/resourceGroups/vm-resources/providers/Microsoft.Network/networkInterfaces/vm01-nic"]
  resource_group_name   = "vm-resources"
  size                  = "Standard_F2"
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  source_image_reference {
    offer     = "WindowsServer"
    publisher = "MicrosoftWindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }
  depends_on = [
    azurerm_network_interface.res-2,
  ]
}
resource "azurerm_network_interface" "res-2" {
  location            = "westeurope"
  name                = "vm01-nic"
  resource_group_name = "vm-resources"
  ip_configuration {
    name                          = "internal"
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = "/subscriptions/resourceGroups/vm-resources/providers/Microsoft.Network/virtualNetworks/iaas-network/subnets/internal"
  }
  depends_on = [
    azurerm_subnet.res-5,
  ]
}
```

### Terraform plan seal test:
lets run a terraform plan on our recently imported terraform configuration (vm-resources) to verify the import was a success, hopefully you will be greeted by the below message. Don't forget to run terraform init and terraform plan against imported resource group working directory.
```
No changes. Your infrastructure matches the configuration.
```

Thanks for taking time to read this Azure Terrafy guide for Windows.







# azure-terrafy-walkthrough

## Steps
 
 ## Dependencies 
 - Install Terraform https://www.terraform.io/downloads
 - Install Azure Terrafy https://github.com/Azure/aztfy
 - PowerShell 7 https://github.com/PowerShell/PowerShell
 - Azure CLI https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-powershell
 - VSCode https://code.visualstudio.com/
 - Windows Terminal https://github.com/microsoft/terminal

 1. Install and download prerequisites
2. Configure VS Code (Extensions)
3. Configure Terraform \ Azure Terrafy  (Extract Terraform executable to c:\terraform, extract Azure Terrafy to c:\aztfy)
   - To add the Terraform\ Azure Terrafy executable directory's to your PATH variable:

     - Click on the Start menu and search for Settings. Open the Settings app.
     - Select the System icon, then on the left menu, select the About tab. Under Related settings on the right, select Advanced system settings.
     - On the System Properties window, select Environment Variables.
     - Select the PATH variable, then click Edit.
     - Click the New button, then type in the path where the Terraform & Terrafy executable is located.

# Following are completed from the CLI (Windows Terminal) 

# Authenticate to Azure    

### Azure Subscription Configuration
#### Azure CLI
1. az login (login)
2. set azure subscription reference az account set --subscription "my sub"

#### Azure PowerShell
1. Connect-AzAccount
2. Set-AzContext -Subscription "Subscription String"

## Azure Terrafy
Create a new directory and use the tool to generate the supporting Terraform code to recreate all of those resouces:
 - mkdir aztfy_netrunner_demo
 - cd aztfy_netruuner_demo (This selects our newly created Azure Terrafy working directory)
 ### Lets Run Azure Terrafy
 In our working diretory run the following command    
 - aztfy "your resource group name"

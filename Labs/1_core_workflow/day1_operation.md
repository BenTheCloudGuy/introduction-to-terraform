#### Understand the terraform core workflow. (write, plan, apply)

## Terraform Core Workflow

* Write
* Init
* Plan 
* Apply

#### Setup

> Start at root of repo /workspaces/introduction-to-terraform/

```bash
# Setup our contoso working directory
mkdir contoso
cd contoso
touch main.tf
```

---

## Day 1 operation (Create)

> NOTE: For the following commands you'll need to be authenticated to Azure and connected to the subscription you want to deploy to. HINT: Use `az login --use-device-code` and `az account set --subscription {YOURSUBSCRIPTIONID}`

```bash
az login --use-device-code
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code F2K7ZDN2U to authenticate.
```

**1. Write**

* Open `main.tf` on the editor and paste in the below code. 

    * If you are in `cloud shell`, you can type **`code .`** and select `main.tf` or simply **`code main.tf`** in terminal to bring up the editor.

```terraform
# Specifiy the provider and version
terraform {
    required_providers {
        azurerm = {
            source  = "hashicorp/azurerm"
            version = "~>3.112.0"
        }
    }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
    features {}
}

# Create the very first resource
resource "azurerm_resource_group" "contoso_rg" {
    name = "contoso_rg"
    location = "WestUS 3"
}
```

* Take a quick look at above code and understand what it does.
* Save `main.tf` (`ctrl + s` should work on cloud shell)

---

**2. Init**
* From terminal, (shortcut `ctrl + '` on cloud shell or vs code)

```bash
# init
terraform init
```

Take a look at what's been created

1. `.terraform.lock.hcl` - Lock file to record the provider selections it made above. 
   ```bash
   # To display dotfiles
   ls -a
   ```

    To upgrade to newer version of the provider in the future, will require a **`terraform init --upgrade`**, which will then also update the lock file. This prevents accidental version bump following a change to `main.tf`. 
    
    Ensure `.terraform.lock.hcl` is version controlled.

2. `.terraform` directory - Contains provider itself that got installed based on the version specified in `main.tf`. 

---
**3. Plan**

```bash
# plan. Below command will generate an execution plan.
# Take a few minutes to go through the terminal output and see what changes will be applied

terraform plan
```  

You should see something like below
    
```bash
▶ terraform plan         

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_resource_group.contoso_rg will be created
  + resource "azurerm_resource_group" "contoso_rg" {
      + id       = (known after apply)
      + location = "westus3"
      + name     = "contoso_rg"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

---

**4. Apply**

```bash
# apply
terraform apply
```

> When prompted to enter a value, type **`yes`** to approve


The terminal output should say something like below

```bash
azurerm_resource_group.contoso_rg: Creating...

azurerm_resource_group.contoso_rg: Creation complete after 1s [id=/subscriptions/.../resourceGroups/contoso_rg]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

> Note that `terraform.tfstate` file has been created locally upon our first apply. State management is a topic on its own and we'll cover it separately. 

---

**5. Verify**

Verify that the resource has been created, either via `Azure Portal` or using `az cli` and `jq` on `cloudshell`

```bash
az group list | jq '.[] | select(.name == "contoso_rg")'

## You should see something similiar to the below.
{
  "id": "/subscriptions/.../resourceGroups/contoso_rg",
  "location": "westus3",
  "managedBy": null,
  "name": "contoso_rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": {},
  "type": "Microsoft.Resources/resourceGroups"
}
```

*  Take a few minutes to understand what `terraform` generated during `apply`

```bash
# Below should display the current state of your terraform managed infrastructure    
terraform show 

#or specifiy the state file name explicitly
terraform show terraform.tfstate
```
> The terraform `show` command is used to provide human-readable output from a state or plan file. See: https://www.terraform.io/docs/commands/show.html


> **Important Note**: 
Besides information about terrafform-managed-resources, `tfstate` will often contain sensitive information and therefore must be kept be very secure. You will see that it's in `.gitignore` to make sure it's not accidentally checked into version control. We'll cover more on State Management later.
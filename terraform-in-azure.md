# Terraform Modules Notes (Microsoft Azure)

## Introduction to Terraform Modules
Terraform modules are reusable configurations that help organize and encapsulate infrastructure components, promoting modularity and maintainability in Infrastructure as Code (IaC).

### Key Benefits of Using Modules:
- **Reusability**: Avoid duplication by defining reusable components.
- **Maintainability**: Easier to manage and update infrastructure.
- **Scalability**: Modules help in scaling infrastructure by defining repeatable patterns.
- **Consistency**: Ensures standardization across multiple environments.

## Structure of a Terraform Module
A typical Terraform module consists of the following files:

```
my-module/
  ├── main.tf       # Defines resources and configurations
  ├── variables.tf  # Declares input variables
  ├── outputs.tf    # Defines output values
  ├── providers.tf  # Specifies provider configurations
  ├── README.md     # Documentation for the module
```

### Explanation of Files:
- **main.tf**: Contains the core resources and infrastructure definitions.
- **variables.tf**: Defines input variables to parameterize module usage.
- **outputs.tf**: Specifies output values that can be used by other modules or Terraform configurations.
- **providers.tf**: Defines required provider configurations (optional in child modules).
- **README.md**: Documents how to use the module.

## Using Terraform Modules
### 1. **Local Module Usage**
Modules can be referenced from local directories:
```hcl
module "network" {
  source = "./modules/network"
  network_name = "my-vnet"
}
```

### 2. **Remote Module Usage**
Modules can be fetched from Terraform Registry, GitHub, or other repositories:
```hcl
module "vnet" {
  source  = "Azure/vnet/azurerm"
  version = "3.0.0"
  vnet_name = "my-vnet"
}
```

### 3. **Module Input Variables**
Modules use input variables to allow customization:
```hcl
variable "vnet_name" {
  type    = string
  default = "my-vnet"
}
```

Usage in a module:
```hcl
resource "azurerm_virtual_network" "main" {
  name                = var.vnet_name
  location            = "East US"
  resource_group_name = "my-resource-group"
  address_space       = ["10.0.0.0/16"]
}
```

### 4. **Module Output Variables**
Modules can expose outputs for other configurations to use:
```hcl
output "vnet_id" {
  value = azurerm_virtual_network.main.id
}
```

### 5. **Using Multiple Providers in a Module**
Terraform modules can use multiple providers to manage infrastructure across different Azure subscriptions or regions. This is done by explicitly defining provider configurations and passing them into the module.

#### Example: Multi-Provider Usage in a Module
```hcl
provider "azurerm" {
  alias           = "subscription1"
  subscription_id = "00000000-0000-0000-0000-000000000000"
  features {}
}

provider "azurerm" {
  alias           = "subscription2"
  subscription_id = "11111111-1111-1111-1111-111111111111"
  features {}
}

module "multi_subscription_storage" {
  source = "./modules/storage"
  providers = {
    azurerm.subscription1 = azurerm.subscription1
    azurerm.subscription2 = azurerm.subscription2
  }
}
```

#### `modules/storage/main.tf`
```hcl
resource "azurerm_storage_account" "primary" {
  provider                = azurerm.subscription1
  name                    = "primarystorageacct"
  resource_group_name     = "rg-primary"
  location                = "East US"
  account_tier            = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_account" "secondary" {
  provider                = azurerm.subscription2
  name                    = "secondarystorageacct"
  resource_group_name     = "rg-secondary"
  location                = "West US"
  account_tier            = "Standard"
  account_replication_type = "LRS"
}
```

This allows resources to be managed across multiple Azure subscriptions.

## Best Practices for Terraform Modules
### 1. **Use Meaningful and Consistent Naming**
   - Choose descriptive names for resources and variables.

### 2. **Keep Modules Focused**
   - A module should serve a single responsibility, such as networking or database provisioning.

### 3. **Document Your Module**
   - Provide a `README.md` with usage examples and descriptions of input/output variables.

### 4. **Use Versioning for Stability**
   - Pin module versions to avoid breaking changes.

### 5. **Separate Environment-Specific Configurations**
   - Use different variable files for staging, production, etc.

### 6. **Follow DRY (Don't Repeat Yourself) Principles**
   - Reuse modules instead of duplicating code.

## Example: Creating and Using a Simple Terraform Module
### Step 1: Define a Simple VNet Module
#### `modules/network/main.tf`
```hcl
resource "azurerm_virtual_network" "main" {
  name                = var.vnet_name
  location            = "East US"
  resource_group_name = "my-resource-group"
  address_space       = ["10.0.0.0/16"]
}
```

#### `modules/network/variables.tf`
```hcl
variable "vnet_name" {
  type = string
}
```

#### `modules/network/outputs.tf`
```hcl
output "vnet_id" {
  value = azurerm_virtual_network.main.id
}
```

### Step 2: Use the Module in Root Configuration
#### `main.tf`
```hcl
module "network" {
  source    = "./modules/network"
  vnet_name = "production-vnet"
}

output "vnet_id" {
  value = module.network.vnet_id
}
```

## Conclusion
Terraform modules provide an efficient way to manage and scale infrastructure by promoting reusability and maintainability. By following best practices and structuring modules effectively, teams can ensure consistency, flexibility, and reliability in their Terraform deployments on Microsoft Azure.


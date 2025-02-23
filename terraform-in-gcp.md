# Terraform Modules Notes (Google Cloud Platform)

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
  network_name = "my-vpc"
}
```

### 2. **Remote Module Usage**
Modules can be fetched from Terraform Registry, GitHub, or other repositories:
```hcl
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "5.0.0"
  network_name = "my-vpc"
}
```

### 3. **Module Input Variables**
Modules use input variables to allow customization:
```hcl
variable "network_name" {
  type    = string
  default = "my-vpc"
}
```

Usage in a module:
```hcl
resource "google_compute_network" "main" {
  name                    = var.network_name
  auto_create_subnetworks = true
}
```

### 4. **Module Output Variables**
Modules can expose outputs for other configurations to use:
```hcl
output "network_id" {
  value = google_compute_network.main.id
}
```

### 5. **Using Multiple Providers in a Module**
Terraform modules can use multiple providers to manage infrastructure across different cloud projects or regions. This is done by explicitly defining provider configurations and passing them into the module.

#### Example: Multi-Provider Usage in a Module
```hcl
provider "google" {
  alias   = "project1"
  project = "my-project-1"
  region  = "us-central1"
}

provider "google" {
  alias   = "project2"
  project = "my-project-2"
  region  = "us-west1"
}

module "multi_project_buckets" {
  source         = "./modules/bucket"
  providers = {
    google.project1 = google.project1
    google.project2 = google.project2
  }
}
```

#### `modules/bucket/main.tf`
```hcl
resource "google_storage_bucket" "primary" {
  provider = google.project1
  name     = "primary-bucket-project1"
  location = "US"
}

resource "google_storage_bucket" "secondary" {
  provider = google.project2
  name     = "secondary-bucket-project2"
  location = "US"
}
```

This allows resources to be managed across multiple Google Cloud projects.

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
### Step 1: Define a Simple VPC Module
#### `modules/network/main.tf`
```hcl
resource "google_compute_network" "main" {
  name                    = var.network_name
  auto_create_subnetworks = true
}
```

#### `modules/network/variables.tf`
```hcl
variable "network_name" {
  type = string
}
```

#### `modules/network/outputs.tf`
```hcl
output "network_id" {
  value = google_compute_network.main.id
}
```

### Step 2: Use the Module in Root Configuration
#### `main.tf`
```hcl
module "network" {
  source       = "./modules/network"
  network_name = "production-network"
}

output "network_id" {
  value = module.network.network_id
}
```

## Conclusion
Terraform modules provide an efficient way to manage and scale infrastructure by promoting reusability and maintainability. By following best practices and structuring modules effectively, teams can ensure consistency, flexibility, and reliability in their Terraform deployments on Google Cloud Platform.


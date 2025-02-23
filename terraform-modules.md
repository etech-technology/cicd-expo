# Terraform Modules Notes

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
  vpc_cidr = "10.0.0.0/16"
}
```

### 2. **Remote Module Usage**
Modules can be fetched from Terraform Registry, GitHub, or other repositories:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.19.0"
  cidr    = "10.0.0.0/16"
}
```

### 3. **Module Input Variables**
Modules use input variables to allow customization:
```hcl
variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}
```

Usage in a module:
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
}
```

### 4. **Module Output Variables**
Modules can expose outputs for other configurations to use:
```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

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
#### `modules/vpc/main.tf`
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags = {
    Name = var.vpc_name
  }
}
```

#### `modules/vpc/variables.tf`
```hcl
variable "vpc_cidr" {
  type = string
}

variable "vpc_name" {
  type = string
  default = "my-vpc"
}
```

#### `modules/vpc/outputs.tf`
```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Step 2: Use the Module in Root Configuration
#### `main.tf`
```hcl
module "vpc" {
  source   = "./modules/vpc"
  vpc_cidr = "10.1.0.0/16"
  vpc_name = "production-vpc"
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

## Sample Project: Building a Custom AWS VPC and EC2 Instance Using Modules Versioned in GitHub
### Step 1: Define the VPC Module (Hosted on GitHub)
#### `vpc/main.tf`
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags = {
    Name = var.vpc_name
  }
}
```
#### `vpc/outputs.tf`
```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Step 2: Define the EC2 Module (Hosted on GitHub)
#### `ec2/main.tf`
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
}
```
#### `ec2/outputs.tf`
```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

### Step 3: Use the Modules in a Terraform Project
#### `main.tf`
```hcl
module "vpc" {
  source   = "git::https://github.com/user/terraform-vpc-module.git?ref=v1.0"
  vpc_cidr = "10.2.0.0/16"
  vpc_name = "custom-vpc"
}

module "ec2" {
  source         = "git::https://github.com/user/terraform-ec2-module.git?ref=v1.0"
  ami_id         = "ami-12345678"
  instance_type  = "t2.micro"
  subnet_id      = module.vpc.vpc_id
}

output "vpc_id" {
  value = module.vpc.vpc_id
}

output "ec2_instance_id" {
  value = module.ec2.instance_id
}
```

## Conclusion
Terraform modules provide an efficient way to manage and scale infrastructure by promoting reusability and maintainability. By following best practices and structuring modules effectively, teams can ensure consistency, flexibility, and reliability in their Terraform deployments.


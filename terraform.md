# Terraform on Azure: A Comprehensive Crash Course
*Last updated: June 7, 2025*

This crash course is designed for professionals preparing for the HashiCorp Certified: Terraform Associate certification while working with Microsoft Azure.

## Table of Contents
- [Introduction to Terraform](#introduction-to-terraform)
- [Installation and Setup](#installation-and-setup)
- [Terraform Basics](#terraform-basics)
- [Terraform Configuration Language](#terraform-configuration-language)
- [Azure Provider Configuration](#azure-provider-configuration)
- [State Management](#state-management)
- [Variables and Outputs](#variables-and-outputs)
- [Resource Dependencies](#resource-dependencies)
- [Modules](#modules)
- [Provisioners](#provisioners)
- [Functions and Expressions](#functions-and-expressions)
- [Meta-Arguments](#meta-arguments)
- [Workspace Management](#workspace-management)
- [State Operations](#state-operations)
- [Terraform Cloud and Enterprise Features](#terraform-cloud-and-enterprise-features)
- [Testing and Debugging](#testing-and-debugging)
- [Best Practices](#best-practices)
- [Practical Examples](#practical-examples)

## Introduction to Terraform

### What is Terraform?
Terraform is an open-source **Infrastructure as Code (IaC)** tool created by HashiCorp. It allows you to define and provision infrastructure resources using a declarative configuration language called HashiCorp Configuration Language (HCL). Unlike traditional scripting, Terraform enables you to describe your desired infrastructure state, and then automatically determines the actions needed to achieve that state.

Terraform works by creating and managing resources through provider APIs. For Azure, the Azure Resource Manager API is used, allowing Terraform to manage virtually any Azure service. The tool excels at managing complex multi-tier architectures across different cloud providers and services.

### Core Benefits
- **Platform Agnostic**: Works with multiple cloud providers (AWS, Azure, GCP) and services simultaneously, allowing you to manage hybrid or multi-cloud environments with a single tool and consistent workflow.
- **Declarative Syntax**: You specify the desired end-state, not the step-by-step process. This approach reduces errors and makes your infrastructure self-documenting.
- **State Management**: Tracks resource states to manage infrastructure effectively, enabling Terraform to understand which resources exist, their configurations, and how to make changes without disruption.
- **Execution Plans**: Shows changes before they're applied, allowing you to review exactly what Terraform will do, reducing risk and increasing confidence.
- **Resource Graph**: Creates a dependency graph for parallel resource creation, which optimizes deployment time by creating non-dependent resources simultaneously.

### Terraform vs. Other IaC Tools
Understanding how Terraform compares to alternatives helps you choose the right tool for your specific needs:

| Tool | Primary Focus | Language | State Management | Key Differentiators |
|------|--------------|----------|------------------|---------------------|
| Terraform | Multi-cloud infrastructure provisioning | HCL | External state file | Provider-agnostic, strong community support, mature ecosystem |
| Azure ARM Templates | Azure-only resources | JSON | Azure backend | Native Azure integration, tight Azure DevOps integration |
| Azure Bicep | Azure-only resources (improved ARM) | Bicep DSL | Azure backend | Improved ARM syntax, better modularity, transpiles to ARM |
| Ansible | Configuration management | YAML | Agentless | Excels at software configuration and orchestration rather than infrastructure creation |
| Pulumi | Infrastructure as actual code | Multiple programming languages | State backends | Uses familiar programming languages, better abstraction capabilities |

When working in Azure specifically, Terraform offers several advantages:
- Provider consistency across multiple clouds if you have hybrid deployments
- Easier to learn language compared to ARM templates
- Better handling of dependencies through explicit declarations
- Robust state management that can be version-controlled

## Installation and Setup

### Installing Terraform on Linux
Terraform installation is straightforward on Linux systems. You download the binary, extract it, and place it in your system PATH. The following steps walk you through the process in detail:

```bash
# Download the latest version from HashiCorp
curl -O https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip

# Unzip the package
unzip terraform_1.7.0_linux_amd64.zip

# Move to a directory in your PATH
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version
```

Alternative installation methods include:

- Using **tfenv** for managing multiple Terraform versions:
  ```bash
  git clone https://github.com/tfutils/tfenv.git ~/.tfenv
  echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.zshrc
  source ~/.zshrc
  tfenv install 1.7.0
  tfenv use 1.7.0
  ```

- Using package managers (on Ubuntu/Debian):
  ```bash
  wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
  echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  sudo apt update
  sudo apt install terraform
  ```

### Installing Azure CLI
Since you're focusing on Azure with Terraform, installing the Azure CLI is essential for authentication and interaction with Azure resources:

```bash
# Install Azure CLI on Ubuntu/Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify installation
az --version

# Login to Azure
az login
```

After login, Azure CLI caches your credentials locally, which Terraform can use for authentication. For non-interactive environments (like CI/CD pipelines), you'll need to use service principals instead.

```bash
# Create a service principal for Terraform
az ad sp create-for-rbac --name "TerraformSP" --role Contributor --scope /subscriptions/YOUR_SUBSCRIPTION_ID

# Output will include appId (client_id), password (client_secret), and tenant
```

### Setting Up Development Environment
A proper environment setup greatly improves your Terraform workflow:

- **VS Code**: Install the HashiCorp Terraform extension for syntax highlighting, validation, and IntelliSense features. The extension supports:
  - Syntax highlighting and validation
  - Automatic formatting on save
  - Hover information for resource attributes
  - Code completion for resource types and properties
  - Navigate to resource definitions and references

- **Authentication**: Configure Azure credentials for Terraform using one of these methods:
  - Environment variables: `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`
  - Azure CLI authentication (simplest for development)
  - Managed Identities (for resources running in Azure)
  - Service Principal with Client Certificate

- **Workspace**: Set up project structure for Terraform configurations:
  ```
  project/
  ├── main.tf           # Main resource definitions
  ├── variables.tf      # Input variable declarations
  ├── outputs.tf        # Output value declarations
  ├── terraform.tfvars  # Actual variable values (git-ignored for sensitive values)
  └── providers.tf      # Provider configurations
  ```

## Terraform Basics

### Terraform Workflow
The Terraform workflow follows a logical sequence that helps manage infrastructure safely and predictably:

1. **Write** configuration files (`.tf`)
   - Define your infrastructure resources in HCL syntax
   - Specify providers, variables, and resource dependencies
   - Organize code into logical modules when appropriate

2. **Initialize** the working directory (`terraform init`)
   - Downloads required provider plugins
   - Sets up backend for storing state
   - Initializes modules referenced in configuration
   - Creates `.terraform` directory to store this configuration

3. **Plan** the changes (`terraform plan`)
   - Creates an execution plan showing what actions Terraform will take
   - Compares the current state with desired configuration
   - Shows resources to be created, modified, or destroyed
   - Allows reviewing changes before implementation
   - Can be saved to a file with `-out=plan.tfplan`

4. **Apply** the changes (`terraform apply`)
   - Executes the actions proposed in the plan
   - Creates, updates, or deletes resources as needed
   - Updates the state file to reflect new resource states
   - Shows a summary of changes made
   - Can apply a previously saved plan with `terraform apply plan.tfplan`

5. **Destroy** resources when no longer needed (`terraform destroy`)
   - Removes all resources managed by Terraform in the configuration
   - Updates state file to reflect changes
   - Can target specific resources using `-target` flag

### Terraform Files
Understanding the different file types is crucial for effective Terraform usage:

- **Configuration files** (`.tf`): Contains resource definitions, providers, variables, and outputs. These are the primary files you'll write and edit.
  - `main.tf`: Main resource definitions
  - `variables.tf`: Variable declarations
  - `outputs.tf`: Output value declarations
  - `providers.tf`: Provider configurations
  - `versions.tf`: Terraform and provider version constraints

- **Variable files** (`.tfvars`): Contains actual values for declared variables, allowing different environments to use the same code with different values.
  - `terraform.tfvars`: Automatically loaded
  - `*.auto.tfvars`: Automatically loaded
  - Custom files loaded with `-var-file=file.tfvars`

- **State files** (`.tfstate`): Contains detailed information about your infrastructure resources, including resource attributes, dependencies, and metadata. Critical for Terraform's operation but should never be edited manually.
  - Contains sensitive information (consider encryption)
  - Should be stored in secure, shared location for teams
  - Local state is stored in `terraform.tfstate`
  - Backup created as `terraform.tfstate.backup`

- **Lock files** (`.terraform.lock.hcl`): Dependency lock file that records the exact provider versions used, ensuring consistency across team members and environments.
  - Should be committed to version control
  - Updated when providers change
  - Ensures reproducible builds

### Basic Commands
Terraform provides a comprehensive CLI with numerous commands. Here are the essential commands with detailed explanations:

```bash
# Initialize a new or existing Terraform configuration
terraform init
# This downloads providers, sets up backend, and prepares modules
# Options:
#   -upgrade: Update all providers to latest version
#   -reconfigure: Reconfigure the backend, useful when changing backend type
#   -backend-config=path: Load backend configuration from file

# Show execution plan
terraform plan
# This creates an execution plan without making changes
# Options:
#   -out=path: Save the plan to a file for later execution
#   -var 'name=value': Set a variable value
#   -var-file=path: Load variable values from file
#   -target=resource: Plan only for specific resource

# Apply changes
terraform apply
# This executes the plan and makes actual infrastructure changes
# Options:
#   -auto-approve: Skip interactive approval
#   Same options as plan for variables and targeting

# Destroy resources
terraform destroy
# Removes all resources managed in this configuration
# Options:
#   -target=resource: Destroy only specific resource
#   -auto-approve: Skip confirmation prompt

# Format configuration files
terraform fmt
# Automatically formats code for consistency
# Options:
#   -recursive: Format files in subdirectories
#   -check: Return error code if files not formatted
#   -write=false: Don't overwrite files (just check)

# Validate configuration
terraform validate
# Checks configuration for syntax errors and internal consistency
# Does not check against real provider APIs

# Show state
terraform show
# Displays human-readable output of state or plan
# Options:
#   path: Path to saved plan file to display

# Output specific value
terraform output [output_name]
# Shows output values from state
# Options:
#   -json: Print output in JSON format
#   -raw: Print the raw string value

# Workspace management
terraform workspace list    # List workspaces
terraform workspace new     # Create a new workspace
terraform workspace select  # Select a workspace
terraform workspace delete  # Delete a workspace
```

These commands form the foundation of your daily interaction with Terraform. Understanding each one and their options will significantly improve your efficiency when managing infrastructure.

## Terraform Configuration Language

### Basic Syntax
HashiCorp Configuration Language (HCL) is designed to be both human-readable and machine-friendly. Here's a breakdown of its key syntactical elements:

```hcl
# Provider configuration
# Providers are plugins that Terraform uses to interact with APIs
provider "azurerm" {
  features {}  # Provider-specific configuration block
  # Other provider settings like authentication can go here
}

# Resource block
# Format: resource "provider_resourcetype" "local_name" { ... }
resource "azurerm_resource_group" "example" {
  name     = "example-resources"  # Arguments specific to this resource type
  location = "West Europe"        # Values can be hardcoded or referenced
  
  tags = {                        # Maps are defined using curly braces
    Environment = "Development"
    Owner       = "Team A"
  }
}

# Data source block
# Data sources fetch information from the provider without creating anything
data "azurerm_subscription" "current" {}  # Empty block means no filters

# Variable declaration
# Variables allow parameterizing configurations
variable "resource_group_name" {
  type        = string              # Enforces type checking
  description = "Name of the resource group"  # Documentation
  default     = "my-resources"      # Default value if not specified
  
  validation {                      # Optional validation rules
    condition     = length(var.resource_group_name) <= 90
    error_message = "Resource group name must be 90 characters or less."
  }
}

# Output value
# Outputs expose specific values from your configuration
output "resource_group_id" {
  value       = azurerm_resource_group.example.id  # References resource attribute
  description = "The ID of the created resource group"  # Documentation
  sensitive   = false                              # Whether to hide in CLI output
}

# Local values
# For intermediate calculations or to avoid repetition
locals {
  common_tags = {
    Project     = "Terraform Demo"
    Environment = var.environment
  }
  
  # Complex expressions are possible
  is_production = var.environment == "prod" ? true : false
}
```

HCL syntax is block-oriented, with blocks containing arguments in key-value format. Comments use the `#` character. Strings can use double quotes or heredoc syntax for multi-line content. The language supports expressions, functions, and operators to create dynamic configurations.

### Block Types
Terraform configurations are composed of various block types, each serving a specific purpose in your infrastructure definition:

- **Provider**: Configures a specific provider (e.g., Azure)
  - Responsible for creating, reading, updating, and deleting resources
  - Can have multiple instances with different configurations using `alias`
  - Example: `provider "azurerm" { features {} }`

- **Resource**: Defines infrastructure objects to create
  - Core building block of Terraform configurations
  - Each resource type maps to a specific API in the provider
  - Format: `resource "TYPE" "NAME" { ... }`
  - Example: `resource "azurerm_virtual_network" "main" { ... }`

- **Data Source**: Fetches data from existing infrastructure
  - Read-only queries to infrastructure
  - Used to reference resources not managed by current Terraform configuration
  - Format: `data "TYPE" "NAME" { ... }`
  - Example: `data "azurerm_resource_group" "existing" { name = "my-existing-rg" }`

- **Variable**: Defines input variables
  - Parameterizes your configurations
  - Can enforce type constraints and validation rules
  - Can have default values or be required
  - Example: `variable "location" { type = string }`

- **Output**: Defines output values
  - Exposes specific values from your configuration
  - Can be used in module composition or to show important information
  - Example: `output "vm_ip" { value = azurerm_public_ip.example.ip_address }`

- **Module**: References reusable configuration components
  - Encapsulates a set of resources for reuse
  - Can accept input variables and produce outputs
  - Example: `module "network" { source = "./modules/network" }`

- **Locals**: Defines local variables
  - For intermediate calculations or to avoid repetition
  - Only usable within the configuration that defines them
  - Example: `locals { name_prefix = "${var.project}-${var.environment}" }`

- **Terraform**: Configures Terraform behavior and backend
  - Sets required Terraform version
  - Configures state storage backend
  - Defines required provider versions
  - Example: `terraform { required_providers { azurerm = { version = "~> 3.0" } } }`

### Resource Addressing
Resource addressing is how you reference resources within Terraform configurations. The addressing syntax is used in expressions, data sources, dependencies, and command-line operations:

- **Generic Pattern**: `<RESOURCE_TYPE>.<NAME>`
  - Used for resources in the root module
  - Example: `azurerm_resource_group.example`
  - Usage: `${azurerm_resource_group.example.location}`

- **Module Resources**: `module.<MODULE_NAME>.<RESOURCE_TYPE>.<NAME>`
  - References resources within modules
  - Example: `module.network.azurerm_subnet.database`
  - Usage: `${module.network.azurerm_subnet.database.id}`

- **Count Index**: `<RESOURCE_TYPE>.<NAME>[<INDEX>]`
  - For resources created with `count` meta-argument
  - Example: `azurerm_subnet.private[0]`
  - Usage: `${azurerm_subnet.private[0].id}`

- **For Each Key**: `<RESOURCE_TYPE>.<NAME>["<KEY>"]`
  - For resources created with `for_each` meta-argument
  - Example: `azurerm_subnet.private["backend"]`
  - Usage: `${azurerm_subnet.private["backend"].id}`

- **CLI References**: Used with commands like `terraform state mv`
  - Example: `terraform state mv azurerm_subnet.old_name azurerm_subnet.new_name`

Understanding resource addressing is essential for creating references between resources, managing state, and using Terraform effectively.

## Azure Provider Configuration

### Basic Provider Setup
Setting up the Azure provider correctly is the foundation for all Azure resource management in Terraform. This section details the complete provider configuration process:

```hcl
# Configure the Azure provider
provider "azurerm" {
  features {
    # Optional feature flags that modify provider behavior
    virtual_machine {
      # Delete OS disk automatically when VM is deleted
      delete_os_disk_on_deletion = true
    }
    key_vault {
      # Purge key vault on destroy
      purge_soft_delete_on_destroy = true
    }
  }
  
  # Authentication details (avoid hardcoding in production)
  subscription_id = "your-subscription-id"
  tenant_id       = "your-tenant-id"
  
  # Optional: API management settings
  partner_id = "your-partner-id"  # For partner attribution
  disable_terraform_partner_id = false
  
  # Optional: specify an exact API endpoint
  # environment = "public" # public, usgovernment, german, china
  
  # For production, use managed identities or environment variables instead
}

# Configure the Terraform backend to store state in Azure Storage
terraform {
  # Define required provider versions
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"  # Source registry/namespace/type
      version = "~> 3.0"            # Version constraint (accepts 3.x, not 4.x)
    }
    # Additional providers can be defined here
    random = {
      source  = "hashicorp/random"
      version = ">= 3.1.0"
    }
  }
  
  # Define Terraform core version requirements
  required_version = ">= 1.0.0, < 2.0.0"
  
  # Azure Storage backend configuration
  backend "azurerm" {
    resource_group_name  = "tfstate"           # Resource group containing storage
    storage_account_name = "tfstate023"        # Globally unique storage account
    container_name       = "tfstate"           # Blob container for states
    key                  = "terraform.tfstate" # State file name within container
    
    # Optional: Use SAS token for authentication (instead of using Azure CLI)
    # sas_token = "your-sas-token"
    
    # Optional: Use access key for authentication (instead of using Azure CLI)
    # access_key = "storage-account-access-key"
  }
}
```

When setting up an Azure backend, you'll need to create these resources first before initializing Terraform. Here's a quick script to create the necessary Azure Storage resources:

```bash
#!/bin/bash
RESOURCE_GROUP_NAME="tfstate"
STORAGE_ACCOUNT_NAME="tfstate$RANDOM"
CONTAINER_NAME="tfstate"
LOCATION="eastus"

# Create resource group
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

# Create storage account
az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

# Create blob container
az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME

# Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query '[0].value' -o tsv)

echo "Storage account name: $STORAGE_ACCOUNT_NAME"
echo "Container name: $CONTAINER_NAME"
echo "Access key: $ACCOUNT_KEY"
```
```

### Authentication Methods
Terraform offers several methods to authenticate with Azure. The choice depends on your environment, security requirements, and operational context:

1. **Azure CLI Authentication**
   - Simplest method for local development
   - Uses credentials from `az login`
   - Does not require explicit credentials in configuration
   - Automatically uses the current subscription context
   
```hcl
provider "azurerm" {
  features {}
  # No explicit credentials - uses Azure CLI
}
```

When to use:
- Interactive development environments
- Local testing and development
- When you need to easily switch between subscriptions

2. **Service Principal with Client Secret**
   - Most common method for automation and CI/CD
   - Requires creating a service principal in Azure AD
   - Can be scoped to specific permissions
   - Credentials should be stored securely
   
```hcl
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
  client_id       = var.client_id
  client_secret   = var.client_secret
  tenant_id       = var.tenant_id
}
```

Creating a service principal:
```bash
# Create service principal and assign Contributor role
az ad sp create-for-rbac --name "TerraformSP" \
  --role "Contributor" \
  --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID" \
  --years 1

# Output includes appId (client_id), password (client_secret), tenant
```

When to use:
- CI/CD pipelines
- Automation scripts
- When fine-grained access control is needed

3. **Managed Identity**
   - For resources running inside Azure
   - No credentials to manage or rotate
   - Most secure option when running in Azure
   
```hcl
provider "azurerm" {
  features {}
  use_msi = true
  # Optional: specify which managed identity to use
  # msi_endpoint = "http://169.254.169.254/metadata/identity/oauth2/token"
}
```

When to use:
- Azure DevOps agents running in Azure
- Terraform running on Azure VMs
- Azure Automation runbooks
- Azure Functions running Terraform

4. **Service Principal with Certificate**
   - More secure alternative to client secrets
   - Uses X.509 certificates for authentication
   
```hcl
provider "azurerm" {
  features {}
  subscription_id   = var.subscription_id
  client_id         = var.client_id
  client_certificate_path = "path/to/certificate.pfx"
  client_certificate_password = var.certificate_password
  tenant_id         = var.tenant_id
}
```

When to use:
- High-security environments
- When organizational policy prohibits service principal secrets
- Long-running automation where certificate rotation is preferred

5. **Environment Variables**
   - Keeps credentials out of configuration files
   - Works with any authentication method
   - Standard practice for security

```bash
# Set environment variables
export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
export ARM_CLIENT_SECRET="12345678-0000-0000-0000-000000000000"
export ARM_TENANT_ID="10000000-0000-0000-0000-000000000000"
export ARM_SUBSCRIPTION_ID="20000000-0000-0000-0000-000000000000"

# For managed identity
export ARM_USE_MSI="true"
```

```hcl
# No authentication in the provider block - uses environment variables
provider "azurerm" {
  features {}
}
```

When to use:
- Local development with sensitive credentials
- CI/CD systems with environment secret support
- Keeping configuration files credential-free

Best practices for authentication:
- Never commit credentials to version control
- Use the principle of least privilege when assigning permissions
- Implement credential rotation policies
- Consider using Azure Key Vault to store and retrieve secrets

### Provider Feature Flags
The Azure provider includes a `features` block that allows you to customize provider behavior. These flags can significantly change how resources are managed, especially during destruction operations:

```hcl
provider "azurerm" {
  features {
    # Resource Group settings
    resource_group {
      # Prevents accidental deletion of non-empty resource groups
      prevent_deletion_if_contains_resources = true
    }
    
    # Virtual Machine settings
    virtual_machine {
      # Automatically delete OS disks when VMs are destroyed
      delete_os_disk_on_deletion = true
      # Graceful shutdown before destruction
      graceful_shutdown = false
    }
    
    # Key Vault settings
    key_vault {
      # Permanently delete key vaults when destroyed (bypasses soft-delete)
      purge_soft_delete_on_destroy = true
      # Recover soft-deleted key vaults when encountered
      recover_soft_deleted_key_vaults = true
    }
    
    # API Management settings
    api_management {
      # Recover API Management services when soft-deleted
      recover_soft_deleted_api_managements = true
    }
    
    # Cognitive Account settings
    cognitive_account {
      # Purge soft-deleted accounts on destruction
      purge_soft_delete_on_destroy = true
    }
    
    # Log Analytics settings
    log_analytics_workspace {
      permanently_delete_on_destroy = true
    }
    
    # Application Insights settings
    application_insights {
      disable_generated_rule = false
    }
  }
}
```

Understanding feature flags is crucial for proper resource lifecycle management:

1. **Deletion behavior flags**:
   These control what happens when you run `terraform destroy` on resources that normally have soft-delete protection. For example, `purge_soft_delete_on_destroy = true` for Key Vault completely removes the vault rather than leaving it in a recoverable state.

2. **Prevention flags**:
   These add safety checks to prevent accidental deletions. For instance, `prevent_deletion_if_contains_resources = true` for resource groups prevents deletion if resources still exist inside.

3. **Recovery flags**:
   These determine whether Terraform should attempt to recover soft-deleted resources when it encounters them. For example, `recover_soft_deleted_key_vaults = true` will recover a soft-deleted Key Vault instead of creating a new one.

When to configure feature flags:
- **Development environments**: May use more aggressive deletion settings for easier cleanup
- **Production environments**: Should use more conservative settings with additional safeguards
- **CI/CD pipelines**: May need specific settings to handle ephemeral environments

Best practices:
- Document your chosen feature flags for team understanding
- Consider different settings for different environments
- Be particularly careful with purge settings in production

## State Management

### Local State
By default, Terraform stores state locally in a `terraform.tfstate` file. This JSON file contains detailed information about your deployed infrastructure, metadata, and resource relationships.

**How local state works:**
- Created automatically in your working directory
- Updated after successful `terraform apply` operations
- Backed up to `terraform.tfstate.backup` before changes
- Contains sensitive information (credentials, IPs, etc.)
- Maps real-world resources to your configuration

**Limitations of local state:**
- Not suitable for team environments (concurrent access issues)
- Risk of accidental deletion or corruption
- Difficult to secure sensitive information
- No version history or audit trail
- Must be manually backed up

**When to use local state:**
- Personal projects
- Learning and experimentation
- Isolated development environments
- When other backend options aren't available

### Remote State
For team collaboration and production environments, remote state is essential. Remote state stores the state file in a shared, persisted location accessible to all team members and CI/CD processes.

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    
    # Optional: state locking configuration
    use_azuread_auth     = true    # Use Azure AD for authentication
    subscription_id      = "your-subscription-id"  # Specify subscription
  }
}
```

**Key benefits of remote state:**
1. **Collaboration**: Multiple team members can work with the same infrastructure
2. **Safety**: Reduced risk of state corruption or loss
3. **Security**: Better access controls for sensitive information
4. **Locking**: Prevents concurrent operations that could corrupt state
5. **History**: Some backends provide versioning and history

**Remote state features:**
1. **State Locking**: Prevents concurrent operations by multiple users
2. **Encryption at Rest**: Protects sensitive data stored in state
3. **Access Controls**: Restrict who can read or write state files
4. **Workspaces**: Manage multiple environments using the same configuration

**Initializing with remote state:**
When changing backends or initializing for the first time with a remote backend:

```bash
# Initialize with new backend
terraform init

# If you have existing local state to migrate
terraform init -migrate-state

# Force reconfiguration of backend
terraform init -reconfigure
```

**Remote state best practices:**
1. Create dedicated storage for Terraform state files
2. Implement strict access controls on state backends
3. Enable encryption for state storage
4. Use state locking to prevent concurrent modifications
5. Implement backup strategies for state files

### Backend Types for Azure
Terraform supports multiple backend types, each with its own characteristics. When working with Azure, you have several options:

1. **azurerm**: Azure Blob Storage
   - **Primary choice** for Azure-focused teams
   - Uses Azure Storage Blob for state storage
   - Supports state locking using blob leases
   - Integrates with Azure RBAC for access control
   - Configuration example:
   ```hcl
   terraform {
     backend "azurerm" {
       resource_group_name  = "tfstate"
       storage_account_name = "tfstate023"
       container_name       = "tfstate"
       key                  = "prod.terraform.tfstate"
     }
   }
   ```
   - **Authentication methods**: Azure CLI, Managed Identity, Service Principal, Access Key, SAS Token

2. **remote**: Terraform Cloud/Enterprise
   - Provides enhanced collaboration features
   - Offers run history and policy controls
   - Includes a private registry for modules
   - Configuration example:
   ```hcl
   terraform {
     cloud {
       organization = "my-organization"
       workspaces {
         name = "my-azure-workspace"
       }
     }
   }
   ```
   - **Best for**: Teams requiring governance, approval workflows, or policy-as-code

3. **s3**: AWS S3 (compatible storage)
   - Can use with Azure Blob Storage that implements S3 compatibility layer
   - Useful for multi-cloud environments that standardize on S3 backend
   - Configuration example:
   ```hcl
   terraform {
     backend "s3" {
       bucket                      = "tfstate"
       key                         = "terraform.tfstate"
       region                      = "us-east-1"
       endpoint                    = "https://s3-compatible-storage.example.com"
       skip_credentials_validation = true
       skip_region_validation      = true
     }
   }
   ```
   - **Best for**: Organizations with existing S3 infrastructure or multi-cloud setups

4. **gcs**: Google Cloud Storage
   - For organizations using Google Cloud alongside Azure
   - Configuration example:
   ```hcl
   terraform {
     backend "gcs" {
       bucket      = "tf-state-prod"
       prefix      = "terraform/state"
     }
   }
   ```
   - **Best for**: Multi-cloud environments including GCP

5. **http**: Generic REST client
   - Works with any HTTP endpoint that implements the required API
   - Can integrate with custom state storage solutions
   - Configuration example:
   ```hcl
   terraform {
     backend "http" {
       address        = "http://custom-state-server.example.com/states/terraform_state"
       lock_address   = "http://custom-state-server.example.com/states/terraform_state/lock"
       unlock_address = "http://custom-state-server.example.com/states/terraform_state/lock"
     }
   }
   ```
   - **Best for**: Custom state storage requirements or enterprise solutions

6. **PostgreSQL**: Database backend
   - Uses a PostgreSQL database for state storage
   - Suitable when you already have PostgreSQL infrastructure
   - Configuration example:
   ```hcl
   terraform {
     backend "pg" {
       conn_str = "postgres://user:password@database.example.com/terraform_backend"
     }
   }
   ```
   - **Best for**: Organizations standardized on PostgreSQL

**Choosing the right backend:**
- **Team size**: Larger teams benefit from remote backend with its collaboration features
- **Security requirements**: Consider encryption, access control needs
- **Existing infrastructure**: Leverage what you already have
- **Feature needs**: Consider locking, versioning, and workspace support
- **Operations model**: Self-hosted vs. managed service preference

### State Locking
State locking is a critical mechanism that prevents state corruption when multiple users or processes run Terraform simultaneously. Without locking, concurrent operations could lead to race conditions, partial updates, or corrupted state files.

**How state locking works in Azure:**
- Azure Storage implements locking using blob leases
- When Terraform operations start, it acquires a lease on the state blob
- Other Terraform operations will wait until the lease is released
- If a lock can't be acquired, Terraform returns an error

**Locking behavior:**
- Locks are acquired during operations that modify state (`apply`, `destroy`, etc.)
- Locks are released when the operation completes (success or failure)
- Read-only operations (`plan` with `-refresh=false`, `output`) don't require locks
- Force-unlock is available but should only be used as a last resort

**Lock configuration with Azure backend:**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    
    # Disable state locking (NOT RECOMMENDED)
    # lock_id = "disabled"
  }
}
```

**When locking fails:**
```bash
Error: Error locking state: Error acquiring the state lock: storage: service returned error: 
StatusCode=409, ErrorCode=LeaseAlreadyPresent, ErrorMessage=There is already a lease present.
```

**Force-unlock (use only when necessary):**
```bash
terraform force-unlock LOCK_ID
```

Best practices:
- **NEVER disable state locking** in team environments
- Use shorter running operations to minimize lock time
- Implement CI/CD pipelines to avoid manual intervention
- Have a documented process for handling stuck locks

### Sensitive Data in State
Terraform state files contain complete details of your infrastructure, including sensitive data like passwords, API keys, and connection strings. This poses significant security risks if not properly managed.

**Examples of sensitive data in state:**
- Database passwords and connection strings
- Private keys and certificates
- API tokens and service principal credentials
- IP addresses and network configurations

**Security measures for protecting state:**
1. **Use encrypted remote backends**:
   - Azure Storage offers encryption at rest by default
   - Enable HTTPS-only access for storage accounts
   - Consider customer-managed keys for additional control

2. **Restrict access to state files**:
   - Use Azure RBAC to limit who can access state storage
   - Implement least-privilege access principles
   - Create dedicated service principals for CI/CD pipelines

3. **Mark sensitive outputs**:
   ```hcl
   output "database_password" {
     value     = azurerm_mysql_server.example.administrator_login_password
     sensitive = true  # Won't display in CLI output
   }
   ```

4. **Use external secrets management**:
   - Azure Key Vault for storing and referencing secrets
   - HashiCorp Vault for dynamic secrets
   - External values for highly sensitive credentials

5. **Audit and monitoring**:
   - Enable Azure Storage analytics and logs
   - Monitor access to state storage accounts
   - Set up alerts for unauthorized access attempts

**Azure Key Vault integration example:**
```hcl
# Store secret in Key Vault
resource "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  value        = random_password.password.result
  key_vault_id = azurerm_key_vault.example.id
}

# Reference it in resources
resource "azurerm_mysql_server" "example" {
  name                = "mysql-server"
  # ...other configuration...
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
}
```

**Recommended state security architecture:**
- Dedicated storage account for Terraform state
- Network security controls (private endpoints, firewall rules)
- Encryption with customer-managed keys
- Regular monitoring and access reviews
- Privileged Identity Management for admin access

### State Commands
Terraform offers a comprehensive set of commands for state manipulation. These commands help you manage, inspect, and modify your infrastructure state without directly editing the state file (which should never be done manually).

```bash
# List resources in state
terraform state list
# Lists all resources being tracked in state
# Useful for: Getting an overview of managed resources, validating configuration

# Show details of a specific resource
terraform state show azurerm_virtual_network.example
# Displays detailed attributes of a specific resource
# Useful for: Debugging, retrieving sensitive values, checking current configuration

# Move a resource to another state address
terraform state mv azurerm_virtual_network.old azurerm_virtual_network.new
# Renames a resource or moves it to a module without destroying and recreating it
# Useful for: Refactoring code, reorganizing configuration, module extraction

# Remove a resource from state
terraform state rm azurerm_virtual_network.example
# Removes a resource from Terraform management without destroying it
# Useful for: Handling resources to be managed manually or by another configuration
# Caution: Resource will no longer be managed by Terraform!

# Pull current state
terraform state pull > backup.tfstate
# Downloads and outputs the state from remote backend to a local file
# Useful for: Backups, inspection, troubleshooting

# Push state from a file
terraform state push backup.tfstate
# Uploads a local state file to the configured backend
# Useful for: Restoring from backup, migrations
# Caution: This overwrites remote state! Use with extreme care.
```

**Advanced state management scenarios:**

1. **Resource refactoring without downtime:**
   ```bash
   # Rename a resource without destroying and recreating it
   terraform state mv azurerm_linux_virtual_machine.old azurerm_linux_virtual_machine.new
   
   # Then update your configuration to match the new name
   ```

2. **Moving resources to modules:**
   ```bash
   # Move a resource into a module
   terraform state mv azurerm_subnet.public module.networking.azurerm_subnet.public
   
   # Then refactor your configuration to match
   ```

3. **Import and manage existing resources:**
   ```bash
   # First create the resource block in your configuration
   # Then import the existing resource into state
   terraform import azurerm_resource_group.example /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/existing-rg
   ```

4. **Disaster recovery:**
   ```bash
   # If remote state is corrupted, restore from backup
   terraform state push backup.tfstate
   ```

**State command best practices:**
- **Always backup** state before significant state operations
- Use explicit **resource addresses** to avoid ambiguity
- Run `terraform plan` after state operations to verify intended changes
- Document state manipulations in version control comments
- Consider running in a staging environment first to validate changes

## Variables and Outputs

Variables and outputs are fundamental to creating flexible, reusable Terraform configurations. Variables act as parameters that allow customization, while outputs expose information about created resources.

### Variable Types
Terraform supports various variable types, from simple scalars to complex structured data:

```hcl
# String variable
variable "location" {
  type        = string
  description = "Azure region for resources"
  default     = "East US"
  
  # Validation ensures the variable contains valid values
  validation {
    condition     = contains(["East US", "West US", "Central US"], var.location)
    error_message = "The location must be a valid Azure US region: East US, West US, or Central US."
  }
}

# Number variable
variable "instance_count" {
  type        = number
  description = "Number of VM instances"
  default     = 2
  
  # Validate minimum and maximum values
  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

# Boolean variable
variable "enable_public_ip" {
  type        = bool
  description = "Enable public IP address"
  default     = false
  
  # Boolean validation is usually unnecessary but can enforce security policies
  validation {
    condition     = var.enable_public_ip == false
    error_message = "Public IPs are not allowed in this environment per security policy."
  }
}

# List variable
variable "allowed_ips" {
  type        = list(string)
  description = "List of allowed IP addresses"
  default     = ["10.0.0.1", "10.0.0.2"]
  
  # Validate all IPs are in the correct format
  validation {
    condition     = alltrue([for ip in var.allowed_ips : can(regex("^\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}$", ip))])
    error_message = "All allowed IPs must be valid IPv4 addresses."
  }
}

# Map variable
variable "common_tags" {
  type        = map(string)
  description = "Common resource tags"
  default     = {
    Environment = "Development"
    Owner       = "Terraform"
  }
  
  # Validate required tags
  validation {
    condition     = contains(keys(var.common_tags), "Environment")
    error_message = "The common_tags variable must contain an Environment tag."
  }
}

# Complex type (object)
variable "vm_config" {
  type = object({
    size        = string
    username    = string
    disk_size   = number
    is_public   = bool
    data_disks  = list(string)
  })
  description = "Virtual machine configuration"
  default = {
    size        = "Standard_DS1_v2"
    username    = "adminuser"
    disk_size   = 128
    is_public   = false
    data_disks  = ["disk1", "disk2"]
  }
  
  # Complex validation
  validation {
    condition     = length(var.vm_config.username) >= 6 && length(var.vm_config.username) <= 20
    error_message = "Username must be between 6 and 20 characters."
  }
}

# Tuple with specific types
variable "network_config" {
  type        = tuple([string, number, bool])
  description = "Network configuration (subnet, port, public)"
  default     = ["subnet1", 80, true]
  
  # Validate port range
  validation {
    condition     = var.network_config[1] > 0 && var.network_config[1] <= 65535
    error_message = "Port must be between 1 and 65535."
  }
}

# Any type (flexible but less safe)
variable "custom_configuration" {
  type        = any
  description = "Custom configuration for resources"
  default     = null
}

# Set variable (like list but unique values)
variable "availability_zones" {
  type        = set(number)
  description = "Azure availability zones to use"
  default     = [1, 2, 3]
}
```

**Variable features:**

1. **Type constraints**:
   - **Primitive types**: `string`, `number`, `bool`
   - **Complex types**: `list`, `set`, `map`, `object`, `tuple`
   - **Type modifiers**: `list(string)`, `map(number)`, `set(bool)`
   
2. **Default values**:
   - Used when no value is explicitly provided
   - Can reference other variables if constructed carefully
   
3. **Input validation**:
   - Multiple validations per variable
   - Complex expressions in conditions
   - Custom error messages

4. **Sensitive marking**:
   ```hcl
   variable "db_password" {
     type        = string
     sensitive   = true  # Won't be shown in logs or console output
     description = "Database administrator password"
   }
   ```

5. **Nullable variables**:
   ```hcl
   variable "optional_config" {
     type     = string
     default  = null
     nullable = true  # Explicitly allow null value
   }
   ```

### Variable Definition Precedence (highest to lowest)
Terraform evaluates variable values according to a strict precedence order, which determines which value takes priority when defined in multiple places:

1. **Command-line flags** (`-var` and `-var-file`)
   - Most direct, highest precedence method
   - Example: `terraform apply -var="instance_count=5" -var-file="prod.tfvars"`
   - Useful for: One-off overrides, CI/CD pipeline parameters
   - Multiple `-var-file` flags are processed in order from left to right

2. **`*.auto.tfvars` files** (alphabetical order)
   - Automatically loaded files ending with `.auto.tfvars` or `.auto.tfvars.json`
   - Processed in alphabetical order (e.g., `a.auto.tfvars` before `b.auto.tfvars`)
   - Useful for: Environment-specific settings, team collaboration
   - Example: `staging.auto.tfvars`, `overrides.auto.tfvars`

3. **`terraform.tfvars.json`**
   - JSON format variable definitions
   - Useful for: Machine-generated variable files, integration with JSON-based systems
   - Example:
     ```json
     {
       "location": "eastus",
       "instance_count": 3
     }
     ```

4. **`terraform.tfvars`**
   - Standard variable definitions file
   - Loaded automatically if present
   - Useful for: Common variable values across environments
   - Example:
     ```hcl
     location = "eastus"
     instance_count = 3
     ```

5. **Environment variables** (`TF_VAR_name`)
   - Set in the shell before running Terraform
   - Must be prefixed with `TF_VAR_`
   - Useful for: Secrets, CI/CD systems, containerized environments
   - Example: `export TF_VAR_db_password="securepassword"`

6. **Default values** in variable declarations
   - Defined in the `variable` blocks
   - Used if no higher-precedence value is available
   - Useful for: Safe fallbacks, sensible defaults
   - Example: `default = "eastus"` in the variable declaration

**Understanding this precedence is crucial for:**
- Debugging variable resolution issues
- Setting up flexible configuration systems
- Managing environment-specific configurations
- Implementing secure handling of sensitive variables

**Example of variable resolution:**
```bash
# In variables.tf
variable "environment" {
  type    = string
  default = "development"
}

# In terraform.tfvars
environment = "staging"

# In production.auto.tfvars
environment = "production"

# Command line
terraform apply -var="environment=testing"

# Result: environment = "testing" (command line has highest precedence)
```

### Output Values
Outputs allow you to extract and display specific values from your infrastructure after deployment. They serve multiple purposes: sharing information between modules, providing important details to users, and enabling integration with other systems.

```hcl
# Basic output
output "resource_group_name" {
  value       = azurerm_resource_group.example.name
  description = "The name of the resource group"
  # Default is false - will show in CLI output
}

# Sensitive output (not shown in logs)
output "admin_password" {
  value       = azurerm_windows_virtual_machine.example.admin_password
  description = "The administrator password for the VM"
  sensitive   = true  # Won't be displayed in terminal output or logs
  # Still accessible programmatically and stored in state
}

# Formatted output with function
output "vm_fqdn" {
  value       = "${azurerm_public_ip.example.fqdn}:${var.app_port}"
  description = "The fully qualified domain name and port for the VM"
  # String interpolation creates a formatted string from multiple values
}

# Output with dependency
output "sql_connection_string" {
  value       = "Server=${azurerm_sql_server.example.fully_qualified_domain_name};Database=${azurerm_sql_database.example.name}"
  description = "The connection string for the SQL database"
  depends_on  = [azurerm_sql_firewall_rule.example]  # Explicitly declare a dependency
  # Ensures firewall rule is created before output is calculated
}

# Complex output with multiple values
output "network_information" {
  value = {
    vnet_id      = azurerm_virtual_network.example.id
    subnet_ids   = [for subnet in azurerm_subnet.example : subnet.id]
    dns_servers  = azurerm_virtual_network.example.dns_servers
    address_space = azurerm_virtual_network.example.address_space
  }
  description = "Network configuration details"
  # Returns a map containing multiple related values
}

# Conditional output
output "public_ip" {
  value       = var.create_public_ip ? azurerm_public_ip.example[0].ip_address : "No public IP created"
  description = "The public IP address (if created)"
  # Only returns a meaningful value when public IP is created
}

# Processed output using functions
output "subnet_cidr_blocks" {
  value       = [for subnet in azurerm_subnet.example : subnet.address_prefixes[0]]
  description = "List of subnet CIDR blocks"
  # Transforms a complex object into a simpler list
}
```

**Key features of outputs:**

1. **Value access:**
   - CLI: `terraform output [output_name]`
   - Raw format: `terraform output -raw output_name`
   - JSON format: `terraform output -json`

2. **Security considerations:**
   - Mark sensitive outputs with `sensitive = true`
   - Sensitive outputs are still stored in state
   - Use for values that shouldn't appear in logs

3. **Module integration:**
   - Child module outputs can be referenced in parent modules
   - Enables clean information flow between modules
   ```hcl
   # In child module:
   output "subnet_id" {
     value = azurerm_subnet.example.id
   }
   
   # In parent module:
   module "network" {
     source = "./modules/network"
   }
   
   output "subnet_id" {
     value = module.network.subnet_id
   }
   ```

4. **Data transformation:**
   - Process data before outputting (sorting, filtering, formatting)
   - Use functions to transform data into more useful formats
   ```hcl
   output "vm_names" {
     value = [for vm in azurerm_linux_virtual_machine.example : vm.name]
   }
   ```

5. **Integration use cases:**
   - CI/CD pipelines: Pass outputs to subsequent deployment steps
   - Documentation: Generate configuration information
   - Testing: Validate deployed resources
   - Cross-stack references: Share data between Terraform configurations

## Resource Dependencies

### Implicit Dependencies
Terraform automatically creates a dependency when one resource references attributes of another. This is the most common and preferred way to establish resource dependencies.

```hcl
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# Implicit dependency on azurerm_resource_group.example
resource "azurerm_virtual_network" "example" {
  name                = "example-vnet"
  resource_group_name = azurerm_resource_group.example.name  # Creates implicit dependency
  location            = azurerm_resource_group.example.location  # Creates implicit dependency
  address_space       = ["10.0.0.0/16"]
}
```

**How implicit dependencies work:**
1. When Terraform parses the configuration, it identifies attribute references
2. It builds a dependency graph based on these references
3. During execution, it ensures dependent resources are created first
4. Changes to referenced resources trigger reevaluation of dependent resources

**Benefits of implicit dependencies:**
- **Self-documenting**: The code clearly shows relationships between resources
- **Automatic**: No need for additional syntax or declarations
- **Precise**: Dependencies are created only where needed
- **Maintainable**: Changing resource names automatically updates dependencies

**Common implicit dependency patterns:**

1. **Parent-child resource relationships**:
```hcl
# Parent resource
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# Child resource with implicit dependency
resource "azurerm_app_service_plan" "example" {
  name                = "example-app-service-plan"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  kind                = "Linux"
  reserved            = true
  
  sku {
    tier = "Basic"
    size = "B1"
  }
}
```

2. **Network resource relationships**:
```hcl
# Networking resources with sequential dependencies
resource "azurerm_virtual_network" "example" {
  name                = "example-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

resource "azurerm_subnet" "example" {
  name                 = "example-subnet"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.example.id
    private_ip_address_allocation = "Dynamic"
  }
}
```

3. **Security dependencies**:
```hcl
# Key Vault access policies with implicit dependencies
resource "azurerm_key_vault" "example" {
  name                = "examplekeyvault"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"
}

resource "azurerm_key_vault_access_policy" "example" {
  key_vault_id = azurerm_key_vault.example.id  # Implicit dependency
  tenant_id    = data.azurerm_client_config.current.tenant_id
  object_id    = azurerm_user_assigned_identity.example.principal_id

  key_permissions = [
    "Get", "List", "Create", "Delete", "Recover"
  ]
}
```

### Explicit Dependencies
Use `depends_on` when a dependency isn't captured through attribute references. This is necessary when resources have a functional dependency that isn't reflected in configuration attributes.

```hcl
# Azure Key Vault
resource "azurerm_key_vault" "example" {
  name                = "example-keyvault"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku_name            = "standard"
  tenant_id           = data.azurerm_client_config.current.tenant_id
}

# Key Vault Access Policy
resource "azurerm_key_vault_access_policy" "example" {
  key_vault_id = azurerm_key_vault.example.id  # Implicit dependency on the key vault
  tenant_id    = data.azurerm_client_config.current.tenant_id
  object_id    = data.azurerm_client_config.current.object_id
  
  key_permissions = [
    "Get", "List", "Create", "Delete"
  ]
}

# Virtual Machine that needs secrets from the vault
resource "azurerm_linux_virtual_machine" "example" {
  name                = "example-vm"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_B1s"
  admin_username      = "adminuser"
  
  # ... other VM configuration ...
  
  # Explicit dependency - VM shouldn't be created until access policy is in place
  # This dependency is not captured in any attribute reference
  depends_on = [azurerm_key_vault_access_policy.example]
}
```

**When to use explicit dependencies:**

1. **Runtime dependencies**: When a resource needs another resource to be fully provisioned before it can function correctly, even if there's no attribute reference.

2. **Sequential operations**: When resources must be created in a specific order despite having no direct attribute references.

3. **Provisioner dependencies**: When provisioners in one resource depend on another resource being available.

4. **Resource replacement coordination**: When destroying and recreating resources must happen in a specific order.

**Common scenarios requiring explicit dependencies:**

1. **IAM and permissions**:
```hcl
# Role assignment must be complete before VM can access storage
resource "azurerm_role_assignment" "example" {
  scope                = azurerm_storage_account.example.id
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = azurerm_user_assigned_identity.example.principal_id
}

resource "azurerm_linux_virtual_machine" "example" {
  # ... VM configuration ...
  identity {
    type         = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.example.id]
  }
  
  # VM needs to wait until it has storage permissions
  depends_on = [azurerm_role_assignment.example]
}
```

2. **Service dependencies**:
```hcl
# Azure Container Registry needs time to complete before AKS can pull from it
resource "azurerm_container_registry" "example" {
  name                = "exampleacr"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku                 = "Standard"
  admin_enabled       = true
}

resource "azurerm_kubernetes_cluster" "example" {
  name                = "example-aks"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "exampleaks"
  
  # ... other AKS configuration ...
  
  # AKS needs ACR to be fully ready
  depends_on = [azurerm_container_registry.example]
}
```

3. **Networking dependencies**:
```hcl
# Private endpoint requires DNS zone to be fully ready
resource "azurerm_private_dns_zone" "example" {
  name                = "privatelink.blob.core.windows.net"
  resource_group_name = azurerm_resource_group.example.name
}

resource "azurerm_private_endpoint" "example" {
  name                = "example-endpoint"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  subnet_id           = azurerm_subnet.example.id
  
  private_service_connection {
    name                           = "example-connection"
    private_connection_resource_id = azurerm_storage_account.example.id
    is_manual_connection           = false
    subresource_names              = ["blob"]
  }
  
  private_dns_zone_group {
    name                 = "default"
    private_dns_zone_ids = [azurerm_private_dns_zone.example.id]
  }
  
  # Ensure DNS zone is fully provisioned
  depends_on = [azurerm_private_dns_zone.example]
}
```

**Best practices for `depends_on`:**
- Use sparingly and only when necessary
- Prefer implicit dependencies through attribute references
- Document the reason for explicit dependencies with comments
- Consider whether the dependency represents a design issue
- Keep dependency lists focused on direct dependencies only

### Dependency Graph
Terraform builds a dependency graph before execution and creates/updates resources in parallel when possible. This directed acyclic graph (DAG) is a core part of how Terraform determines the order of operations.

**How the dependency graph works:**
1. During the planning phase, Terraform analyzes all resources and their dependencies
2. It constructs a directed graph where nodes are resources and edges are dependencies
3. Resources with no dependencies between them can be provisioned in parallel
4. Terraform uses this graph to maximize parallelism while respecting dependencies

**Visualizing the dependency graph:**
```bash
# Generate graph in DOT format
terraform graph > graph.dot

# Convert to SVG using Graphviz
dot -Tsvg graph.dot -o graph.svg

# For a plan-specific graph
terraform plan -out=tfplan
terraform graph -plan=tfplan > plan_graph.dot
dot -Tsvg plan_graph.dot -o plan_graph.svg

# Different graph types
terraform graph -type=plan       # Plan graph (default)
terraform graph -type=plan-destroy # Destroy plan graph
terraform graph -type=apply      # Apply graph
terraform graph -type=validate   # Validation graph
```

**Installing Graphviz (required for graph visualization):**
```bash
# On Ubuntu/Debian
sudo apt-get install graphviz

# On macOS with Homebrew
brew install graphviz

# On Windows with Chocolatey
choco install graphviz
```

**Understanding the graph components:**
- **Green nodes**: Resources to be created
- **Red nodes**: Resources to be destroyed
- **Yellow nodes**: Resources to be updated in-place
- **Blue nodes**: Data sources to be read
- **Edges (lines)**: Dependencies between resources

**Benefits of understanding the dependency graph:**
1. **Debugging**: Identify unexpected dependencies or cycles
2. **Optimization**: Improve deployment speed by reducing unnecessary dependencies
3. **Refactoring**: Visualize the impact of structural changes
4. **Documentation**: Generate visual representation of infrastructure

**Advanced dependency management:**
1. **Parallelism control**: Adjust the number of concurrent operations
   ```bash
   terraform apply -parallelism=5  # Limit to 5 concurrent operations
   ```

2. **Dependency cycle detection**: Terraform automatically detects circular dependencies
   ```
   Error: Cycle: azurerm_network_interface.example, azurerm_virtual_machine.example
   ```

3. **Targeted operations**: Apply or destroy specific resources and their dependencies
   ```bash
   terraform apply -target=azurerm_virtual_machine.example
   ```

**Complex dependency graph example:**
```
digraph {
  compound = true
  newrank = true

  "provider[\"registry.terraform.io/hashicorp/azurerm\"]" -> "azurerm_resource_group.example"
  "azurerm_resource_group.example" -> "azurerm_virtual_network.example"
  "azurerm_virtual_network.example" -> "azurerm_subnet.example"
  "azurerm_subnet.example" -> "azurerm_network_interface.example"
  "azurerm_network_interface.example" -> "azurerm_linux_virtual_machine.example"
}
```

This graph shows a simple linear dependency chain from resource group to VM, visualizing how Terraform will sequence the creation of these resources.

## Modules

### Module Structure
Modules are self-contained packages of Terraform configurations that are managed as a group. A well-structured module enhances reusability, maintainability, and encapsulation.

A typical Terraform module structure:

```
module/
├── main.tf         # Main resources and primary logic
├── variables.tf    # Input variables declaration and validation
├── outputs.tf      # Output values exposed from the module
├── versions.tf     # Required providers and Terraform versions
├── README.md       # Documentation on module usage and examples
├── providers.tf    # (Optional) Provider configuration if needed
├── data.tf         # (Optional) Data sources separated for clarity
├── locals.tf       # (Optional) Local values for internal use
├── examples/       # (Optional) Example implementations
│   ├── basic/      # Basic usage example
│   └── complete/   # Advanced usage example
└── tests/          # (Optional) Automated tests for the module
    └── fixtures/   # Test configurations
```

**Purpose of each file:**

1. **main.tf**: Contains the primary resources that the module manages. This is where the core infrastructure is defined. For complex modules, you might split this into multiple files like `network.tf`, `compute.tf`, etc.

2. **variables.tf**: Defines all input variables accepted by the module. Each variable should include:
   - Type constraint
   - Description
   - Default value (if applicable)
   - Validation rules (if necessary)

3. **outputs.tf**: Defines values to be exposed by the module. These outputs can be referenced by the calling module or root configuration. Outputs should include:
   - Value to be returned
   - Description
   - Sensitive flag if applicable

4. **versions.tf**: Specifies the required Terraform and provider versions. This ensures compatibility and consistent behavior.
   ```hcl
   terraform {
     required_version = ">= 1.0.0"
     required_providers {
       azurerm = {
         source  = "hashicorp/azurerm"
         version = "~> 3.0"
       }
     }
   }
   ```

5. **README.md**: Documentation that explains:
   - What the module does
   - How to use it (examples)
   - Required and optional inputs
   - Outputs provided
   - Any special considerations or limitations

**Optional components:**

6. **providers.tf**: Typically, child modules should inherit providers from the root module. However, in some cases you might need explicit provider configurations.

7. **data.tf**: Separates data sources for clarity in complex modules.

8. **locals.tf**: Contains local values used within the module for calculations, naming standardization, etc.

9. **examples/**: Provides working examples of module usage, which also serve as documentation.

10. **tests/**: Contains automated tests for the module using frameworks like Terratest.

**Module organization best practices:**

1. **Single responsibility principle**: Each module should do one thing and do it well
2. **Encapsulation**: Hide internal complexity, expose only necessary interfaces
3. **Consistent naming**: Use consistent naming across variables, resources, and outputs
4. **Complete documentation**: Ensure README contains all information needed to use the module
5. **Sensible defaults**: Provide reasonable defaults but allow overrides for flexibility
6. **Validation**: Include validation rules to prevent misuse
7. **Version constraints**: Specify compatible Terraform and provider versions

### Creating a Module
Creating reusable modules is a fundamental skill for effective Terraform usage. Below is a comprehensive example of a Virtual Network module for Azure with detailed comments explaining each part:

```hcl
# versions.tf
# Defines compatibility requirements for Terraform and providers
terraform {
  required_version = ">= 1.0.0, < 2.0.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0.0, < 4.0.0"
    }
  }
}

# variables.tf
# Input variables provide configuration options for the module
variable "resource_group_name" {
  type        = string
  description = "Name of the resource group"
  
  validation {
    condition     = length(var.resource_group_name) >= 3 && length(var.resource_group_name) <= 63
    error_message = "Resource group name must be between 3 and 63 characters."
  }
}

variable "location" {
  type        = string
  description = "Azure location"
}

variable "vnet_name" {
  type        = string
  description = "Name of the virtual network"
  
  validation {
    condition     = length(var.vnet_name) >= 2 && length(var.vnet_name) <= 64
    error_message = "Virtual network name must be between 2 and 64 characters."
  }
}

variable "address_space" {
  type        = list(string)
  description = "Address space for the virtual network"
  default     = ["10.0.0.0/16"]
  
  validation {
    condition     = length(var.address_space) > 0
    error_message = "At least one address space must be provided."
  }
}

variable "subnet_prefixes" {
  type        = list(string)
  description = "List of subnet address prefixes"
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "subnet_names" {
  type        = list(string)
  description = "List of subnet names"
  default     = ["subnet1", "subnet2"]
}

variable "tags" {
  type        = map(string)
  description = "Tags to apply to all resources"
  default     = {}
}

variable "dns_servers" {
  type        = list(string)
  description = "Optional custom DNS servers"
  default     = []
}

variable "subnet_service_endpoints" {
  type        = map(list(string))
  description = "Map of subnet names to service endpoints to enable"
  default     = {}
}

# locals.tf
# Local values for internal calculations and naming conventions
locals {
  # Ensure subnet counts match
  subnet_count = max(length(var.subnet_names), length(var.subnet_prefixes))
  
  # Merge default tags with user-provided tags
  default_tags = {
    ManagedBy = "Terraform"
    Module    = "azure-vnet"
  }
  
  all_tags = merge(local.default_tags, var.tags)
  
  # Default service endpoints if none specified
  default_subnet_service_endpoints = {
    for name in var.subnet_names : name => []
  }
  
  subnet_service_endpoints_merged = merge(local.default_subnet_service_endpoints, var.subnet_service_endpoints)
}

# main.tf
# Core resource definitions
resource "azurerm_virtual_network" "vnet" {
  name                = var.vnet_name
  resource_group_name = var.resource_group_name
  location            = var.location
  address_space       = var.address_space
  dns_servers         = length(var.dns_servers) > 0 ? var.dns_servers : null
  
  tags = local.all_tags
  
  # Lifecycle to prevent accidental deletion
  lifecycle {
    prevent_destroy = false
  }
}

resource "azurerm_subnet" "subnet" {
  count                = local.subnet_count
  name                 = var.subnet_names[count.index]
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [var.subnet_prefixes[count.index]]
  
  # Add service endpoints if configured
  service_endpoints = lookup(
    local.subnet_service_endpoints_merged,
    var.subnet_names[count.index],
    []
  )
  
  # Important: the delegations, service endpoint policies and network security group 
  # associations would typically be handled here or in separate resources
}

# data.tf
# Data sources to look up existing resources if needed
data "azurerm_resource_group" "existing" {
  name = var.resource_group_name
}

# outputs.tf
# Values exposed by the module
output "vnet_id" {
  description = "The ID of the virtual network"
  value       = azurerm_virtual_network.vnet.id
}

output "vnet_name" {
  description = "The name of the virtual network"
  value       = azurerm_virtual_network.vnet.name
}

output "vnet_address_space" {
  description = "The address space of the virtual network"
  value       = azurerm_virtual_network.vnet.address_space
}

output "subnet_ids" {
  description = "The IDs of the created subnets"
  value       = azurerm_subnet.subnet[*].id
}

output "subnet_names" {
  description = "The names of the created subnets"
  value       = azurerm_subnet.subnet[*].name
}

output "subnet_address_prefixes" {
  description = "The address prefixes of the created subnets"
  value       = azurerm_subnet.subnet[*].address_prefixes[0]
}

# Create a map of subnet names to IDs for easier reference
output "subnet_id_map" {
  description = "Map of subnet names to IDs"
  value       = { for i, subnet in azurerm_subnet.subnet : subnet.name => subnet.id }
}
```

**README.md for the module:**

```markdown
# Azure Virtual Network Terraform Module

This Terraform module deploys a Virtual Network in Azure with a configurable number of subnets.

## Features

* Create a virtual network with multiple subnets
* Configure DNS servers
* Enable service endpoints on subnets
* Apply tags to resources

## Usage

```hcl
module "vnet" {
  source              = "./modules/vnet"
  resource_group_name = "my-resources"
  location            = "eastus"
  vnet_name           = "my-vnet"
  address_space       = ["10.0.0.0/16"]
  subnet_names        = ["web", "app", "db"]
  subnet_prefixes     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  
  # Optional configuration
  tags = {
    Environment = "Production"
    Project     = "Example"
  }
  
  dns_servers = ["10.0.0.4", "10.0.0.5"]
  
  subnet_service_endpoints = {
    web = ["Microsoft.KeyVault"]
    db  = ["Microsoft.Sql", "Microsoft.Storage"]
  }
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.0.0, < 2.0.0 |
| azurerm | >= 3.0.0, < 4.0.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| resource_group_name | Name of the resource group | `string` | n/a | yes |
| location | Azure location | `string` | n/a | yes |
| vnet_name | Name of the virtual network | `string` | n/a | yes |
| address_space | Address space for the virtual network | `list(string)` | `["10.0.0.0/16"]` | no |
| subnet_prefixes | List of subnet address prefixes | `list(string)` | `["10.0.1.0/24", "10.0.2.0/24"]` | no |
| subnet_names | List of subnet names | `list(string)` | `["subnet1", "subnet2"]` | no |
| tags | Tags to apply to all resources | `map(string)` | `{}` | no |
| dns_servers | Optional custom DNS servers | `list(string)` | `[]` | no |
| subnet_service_endpoints | Map of subnet names to service endpoints to enable | `map(list(string))` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| vnet_id | The ID of the virtual network |
| vnet_name | The name of the virtual network |
| vnet_address_space | The address space of the virtual network |
| subnet_ids | The IDs of the created subnets |
| subnet_names | The names of the created subnets |
| subnet_address_prefixes | The address prefixes of the created subnets |
| subnet_id_map | Map of subnet names to IDs |
```

**Key practices demonstrated in this module:**

1. **Input validation**: Ensuring valid parameters are provided
2. **Default values**: Sensible defaults for optional parameters
3. **Local variables**: Internal calculations and normalization
4. **Comprehensive outputs**: Exposing all useful resource attributes
5. **Documentation**: Clear README with examples and reference
6. **Tags**: Implementing consistent tagging
7. **Resource mapping**: Creating useful data structures (subnet map)
8. **Defensive coding**: Handling edge cases like mismatched subnet counts

### Using Modules
Modules are called from other Terraform configurations using the `module` block. Understanding how to effectively use modules is essential for building scalable and maintainable infrastructure.

```hcl
module "network" {
  source              = "./modules/vnet"  # Path to module directory
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  vnet_name           = "example-vnet"
  address_space       = ["10.0.0.0/16"]
  subnet_prefixes     = ["10.0.1.0/24", "10.0.2.0/24"]
  subnet_names        = ["web", "database"]
  
  # Optional configuration with module-specific parameters
  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

# Referencing module outputs
# Module outputs are accessed using module.<NAME>.<OUTPUT>
resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = module.network.subnet_ids[0]  # Accessing module output
    private_ip_address_allocation = "Dynamic"
  }
}

# Using module output in other resources 
resource "azurerm_network_security_group" "example" {
  name                = "example-nsg"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

# Using the subnet ID map from module output
resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = module.network.subnet_id_map["web"]
  network_security_group_id = azurerm_network_security_group.example.id
}
```

**Key aspects of module usage:**

1. **Module source specification**:
   - Local path: `source = "./modules/network"`
   - Git repository: `source = "git::https://example.com/network.git"`
   - Terraform Registry: `source = "Azure/network/azurerm"`
   - Version constraint: `version = "3.5.0"`

2. **Input variables**:
   - Passed as arguments to the module
   - Match the variables defined in the module
   - Can include expressions, resource attributes, or other variables

3. **Output usage**:
   - Access using the syntax `module.<NAME>.<OUTPUT>`
   - Can be referenced in other resources
   - Can be exposed as outputs from the root module

4. **Module composition**:
   - Modules can call other modules (nesting)
   - Creates a hierarchy of infrastructure components
   ```hcl
   module "app" {
     source = "./modules/app"
     
     # Pass outputs from one module to another
     subnet_id = module.network.subnet_ids[0]
   }
   ```

5. **Conditional module usage**:
   - Use `count` to conditionally create modules
   ```hcl
   module "optional_component" {
     count  = var.create_component ? 1 : 0
     source = "./modules/component"
     # ...other arguments
   }
   ```

6. **Dynamic module usage**:
   - Use `for_each` to create multiple module instances
   ```hcl
   module "microservice" {
     for_each = var.microservices
     source   = "./modules/microservice"
     
     name     = each.key
     config   = each.value
   }
   ```

7. **Provider configuration**:
   - Modules inherit provider configuration from the caller
   - Can be overridden within modules if needed

**Best practices:**

1. **Keep module calls focused**:
   Each module should do one thing well, and module calls should reflect this.

2. **Prefer explicit dependencies**:
   Make dependencies clear by referencing outputs directly.

3. **Use consistent naming**:
   Name module instances according to their purpose, not implementation.

4. **Document module usage**:
   Comments should explain why a module is used and any non-obvious configurations.

5. **Parameterize environment-specific values**:
   Pass environment variables to make modules reusable across environments.

```hcl
# Example of environment-specific configuration
module "database" {
  source              = "./modules/database"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  
  # Environment-specific settings
  tier                = var.environment == "prod" ? "Premium" : "Basic"
  capacity            = var.environment == "prod" ? 4 : 1
  backup_retention    = var.environment == "prod" ? 35 : 7
  
  tags = {
    Environment = var.environment
  }
}

### Module Sources
Terraform supports retrieving modules from various locations, giving you flexibility in how you organize and share your infrastructure code. Each source type has specific syntax and features:

1. **Local Path**
```hcl
module "network" {
  source = "./modules/network"  # Relative path
}

module "storage" {
  source = "/absolute/path/to/modules/storage"  # Absolute path (not recommended)
}
```
- **Best for**: Local development, modules specific to a single project
- **Features**: Simplest to use, direct file access
- **Limitations**: Not shareable across projects, lacks versioning
- **Initialization**: Fast, no download required

2. **GitHub**
```hcl
# Public repository
module "network" {
  source = "github.com/terraform-azure-modules/azure-network-module"
  # Optional: specify a specific branch, tag, or commit
  # source = "github.com/terraform-azure-modules/azure-network-module?ref=v1.2.0"
}

# Private repository with HTTPS authentication
module "network" {
  source = "git::https://github.com/myorg/private-modules.git//modules/network?ref=v1.0.0"
  # Authentication via HTTPS token in the URL or SSH key
}

# Specific subdirectory in a repository
module "subnet" {
  source = "github.com/terraform-azure-modules/azure-network-module//modules/subnet"
}
```
- **Best for**: Open source modules, public sharing
- **Features**: Widely accessible, familiar workflow
- **Authentication**: SSH keys, PATs
- **Reference options**: branches, tags, commits

3. **Terraform Registry**
```hcl
module "network" {
  source  = "Azure/network/azurerm"  # Format: <NAMESPACE>/<NAME>/<PROVIDER>
  version = "3.5.0"  # Specific version
  # version = "~> 3.0"  # Version constraint (any 3.x version)
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = ">= 3.0.0, < 4.0.0"  # Complex version constraint
}
```
- **Best for**: Community modules, widely used components
- **Features**: Versioning, documentation, input/output discovery
- **Public Registry**: https://registry.terraform.io
- **Private Registry**: Self-hosted or Terraform Cloud/Enterprise

4. **Azure DevOps**
```hcl
module "network" {
  source = "git::https://dev.azure.com/organization/project/_git/repo//modules/network"
  
  # With version tag
  # source = "git::https://dev.azure.com/organization/project/_git/repo//modules/network?ref=v1.0.0"
  
  # With personal access token
  # source = "git::https://username:token@dev.azure.com/organization/project/_git/repo//modules/network"
}
```
- **Best for**: Enterprise environments using Azure DevOps
- **Features**: Integration with Azure DevOps workflows
- **Authentication**: PATs, Azure AD

5. **Bitbucket**
```hcl
module "network" {
  source = "bitbucket.org/myorg/terraform-modules//modules/network"
}
```
- **Best for**: Teams using Bitbucket
- **Authentication**: Similar to GitHub

6. **S3 / Cloud Storage**
```hcl
module "network" {
  source = "s3::https://s3-eu-west-1.amazonaws.com/terraform-modules/network.zip"
}

module "storage" {
  source = "gcs::https://www.googleapis.com/storage/v1/terraform-modules/network.zip"
}

module "azure_module" {
  source = "azurerm::https://storageaccount.blob.core.windows.net/modules/network.zip"
}
```
- **Best for**: Air-gapped environments, pre-packaged modules
- **Features**: Binary distribution, no Git required
- **Format**: Zip archives
- **Authentication**: Cloud provider credentials

7. **HTTP URLs**
```hcl
module "network" {
  source = "https://example.com/modules/network.zip"
}
```
- **Best for**: Simple distribution without Git
- **Features**: Works with any web server
- **Format**: Zip archives
- **Authentication**: HTTP basic auth, headers

8. **Terraform Cloud/Enterprise Private Registry**
```hcl
module "network" {
  source  = "app.terraform.io/myorg/network/azurerm"
  version = "1.0.0"
}
```
- **Best for**: Enterprise environments, private modules
- **Features**: Private hosting, access controls, versioning
- **Authentication**: Terraform Cloud/Enterprise credentials

**Version selection:**
When using remote modules, always specify a version to ensure stability:

```hcl
module "network" {
  source  = "Azure/network/azurerm"
  version = "3.5.0"  # Exact version - most stable
  # version = "~> 3.5.0"  # Patch updates only (3.5.x)
  # version = "~> 3.5"    # Minor updates (3.x)
  # version = ">= 3.0.0, < 4.0.0"  # Range constraint
}
```

**Security considerations:**
- Validate modules from public sources before use
- Consider checksums for HTTP/S3 modules
- Use private module registries for sensitive code
- Implement a review process for external modules
- Pin to specific versions or commits to prevent supply chain attacks

### Module Versioning
Proper version management is crucial for module stability and compatibility. Terraform uses Semantic Versioning (SemVer) for modules, following the MAJOR.MINOR.PATCH format:

```hcl
module "web_server_cluster" {
  source  = "Azure/compute/azurerm"
  version = "~> 3.0"  # Accepts any 3.x version but not 4.0+
}
```

**Version constraint operators:**

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `= 3.0.0` | Exact version match only |
| `!=` | `!= 3.0.0` | Any version except the specified one |
| `>` | `> 3.0.0` | Greater than specified version |
| `>=` | `>= 3.0.0` | Greater than or equal to specified version |
| `<` | `< 4.0.0` | Less than specified version |
| `<=` | `<= 3.10.0` | Less than or equal to specified version |
| `~>` | `~> 3.0` | Any version in the same minor series (3.x) |
| `~>` | `~> 3.0.0` | Any version in the same patch series (3.0.x) |

**Combining constraints:**

```hcl
module "complex_version" {
  source  = "Azure/compute/azurerm"
  version = ">= 3.0.0, < 4.0.0"  # At least 3.0.0 but less than 4.0.0
}
```

**Version selection strategies:**

1. **Exact version** (`= 3.0.0`):
   - Most stable and predictable
   - No risk of unexpected changes
   - Must manually update for fixes and improvements
   ```hcl
   module "stable" {
     source  = "Azure/compute/azurerm"
     version = "3.0.0"  # Exact version match (= is implied)
   }
   ```

2. **Patch updates only** (`~> 3.0.0`):
   - Allows bug fixes (3.0.1, 3.0.2, etc.)
   - Won't include new features or breaking changes
   - Good balance of stability and fixes
   ```hcl
   module "patch_updates" {
     source  = "Azure/compute/azurerm"
     version = "~> 3.0.0"  # Allows 3.0.x but not 3.1.0+
   }
   ```

3. **Minor updates** (`~> 3.0`):
   - Allows new features and bug fixes (3.1.0, 3.2.0, etc.)
   - Should not include breaking changes
   - More automatic updates but slightly higher risk
   ```hcl
   module "minor_updates" {
     source  = "Azure/compute/azurerm"
     version = "~> 3.0"  # Allows 3.x but not 4.0+
   }
   ```

4. **Range constraint** (`>= 3.0.0, < 4.0.0`):
   - Explicitly defines acceptable version range
   - Useful for complex requirements
   - Same effect as `~> 3.0` in this example
   ```hcl
   module "range_constraint" {
     source  = "Azure/compute/azurerm"
     version = ">= 3.0.0, < 4.0.0"
   }
   ```

**Best practices for module versioning:**

1. **Always specify a version** for non-local modules
   ```hcl
   # Incorrect - no version constraint
   module "risky" {
     source = "Azure/compute/azurerm"
   }
   ```

2. **Use conservative constraints** in production
   ```hcl
   # Good for production
   module "production" {
     source  = "Azure/compute/azurerm"
     version = "~> 3.0.0"  # Patch updates only
   }
   ```

3. **Pin exact versions** for maximum stability
   ```hcl
   # Maximum stability
   module "critical" {
     source  = "Azure/compute/azurerm"
     version = "3.0.0"
   }
   ```

4. **Document your version policy** in comments
   ```hcl
   module "web" {
     source  = "Azure/compute/azurerm"
     version = "~> 3.0.0"  # Only accept patch updates for stability
   }
   ```

5. **Update versions as part of your change management process**
   - Schedule regular reviews of module versions
   - Test upgrades in non-production environments
   - Document changes when upgrading versions

6. **Lock file management**
   - Commit `.terraform.lock.hcl` to version control
   - Use `terraform providers lock` to manage provider locks
   - Update locks deliberately with `terraform init -upgrade`

## Provisioners

### Types of Provisioners

Provisioners allow you to execute scripts or actions as part of resource creation or destruction. While generally considered a last resort in Terraform, they provide important capabilities for scenarios where the provider API doesn't offer the required functionality.

1. **local-exec**: Runs a command on the machine running Terraform
```hcl
resource "azurerm_storage_account" "example" {
  name                     = "examplestorage"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  provisioner "local-exec" {
    command = "echo Storage account ${self.name} created with ${self.primary_connection_string} > storage_output.txt"
    
    # Optional parameters
    working_dir = "/tmp"                      # Directory to run command in
    interpreter = ["/bin/bash", "-c"]         # Command interpreter
    environment = {                           # Environment variables
      STORAGE_NAME = self.name
      AZURE_REGION = self.location
    }
    
    # Optional: Only run when something changes (not on every apply)
    when = create     # Options: "create" or "destroy"
  }
}
```

**Local-exec use cases:**
- Record resource information in external systems
- Trigger CI/CD pipelines after infrastructure changes
- Populate databases or configuration stores
- Run local scripts that interact with created resources
- Generate documentation or diagrams from infrastructure

**Advanced local-exec example:**
```hcl
# Run a PowerShell script on Windows
provisioner "local-exec" {
  command     = "Write-Output 'Azure resources created: ${local.resource_count}'"
  interpreter = ["PowerShell", "-Command"]
}

# Use conditional execution
provisioner "local-exec" {
  command = "python ./scripts/register_resource.py --resource-id=${self.id}"
  
  # Only run if a specific condition is met
  environment = {
    SHOULD_REGISTER = var.environment == "production" ? "true" : "false"
  }
}
```

2. **remote-exec**: Runs a command on a remote resource
```hcl
resource "azurerm_linux_virtual_machine" "example" {
  name                  = "example-vm"
  resource_group_name   = azurerm_resource_group.example.name
  location              = azurerm_resource_group.example.location
  size                  = "Standard_B1s"
  admin_username        = "adminuser"
  network_interface_ids = [azurerm_network_interface.example.id]
  
  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  # Configure SSH connection
  connection {
    type        = "ssh"                      # Connection type: ssh or winrm
    host        = self.public_ip_address     # Host to connect to
    user        = "adminuser"                # Login user
    private_key = file("~/.ssh/id_rsa")      # Authentication
    timeout     = "5m"                       # Connection timeout
    agent       = false                      # Don't use SSH agent
  }

  provisioner "remote-exec" {
    # Run commands directly
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx",
      "sudo systemctl start nginx",
      "echo 'Server configured by Terraform' | sudo tee /var/www/html/index.html"
    ]
    
    # Alternative: Run a script
    # script = "scripts/setup.sh"
    
    # Alternative: Run scripts from a directory
    # scripts = ["scripts/install.sh", "scripts/configure.sh"]
  }
}
```

**Remote-exec use cases:**
- Install software on newly created VMs
- Configure applications post-deployment
- Join servers to domains or clusters
- Start services or run initialization routines
- Apply complex configurations that aren't supported by provider APIs

**Advanced remote-exec example:**
```hcl
# Windows VM with WinRM connection
resource "azurerm_windows_virtual_machine" "example" {
  # ... VM configuration ...
  
  connection {
    type     = "winrm"
    host     = self.public_ip_address
    user     = "adminuser"
    password = azurerm_key_vault_secret.vm_password.value
    https    = true
    insecure = true    # Ignore certificate errors (not recommended for production)
    timeout  = "10m"
  }
  
  provisioner "remote-exec" {
    inline = [
      "powershell -Command \"Install-WindowsFeature -Name Web-Server -IncludeAllSubFeature\"",
      "powershell -Command \"New-Item -Path 'C:\\inetpub\\wwwroot\\index.html' -ItemType File -Value '<html><body><h1>Server configured with Terraform</h1></body></html>'\"",
    ]
  }
}
```

3. **file**: Copies files to a remote resource
```hcl
resource "azurerm_linux_virtual_machine" "example" {
  # ... VM configuration ...

  connection {
    type        = "ssh"
    host        = self.public_ip_address
    user        = "adminuser"
    private_key = file("~/.ssh/id_rsa")
  }

  # Upload a single file
  provisioner "file" {
    source      = "app/config.json"           # Local file path
    destination = "/etc/app/config.json"      # Remote destination path
  }
  
  # Upload a directory
  provisioner "file" {
    source      = "app/config/"               # Local directory (with trailing slash)
    destination = "/etc/app/"                 # Remote directory
  }
  
  # Upload generated content
  provisioner "file" {
    content     = templatefile("${path.module}/templates/app.conf.tpl", {
      app_port    = var.app_port
      environment = var.environment
    })
    destination = "/etc/app/app.conf"
  }
  
  # After uploading files, configure permissions
  provisioner "remote-exec" {
    inline = [
      "chmod +x /etc/app/startup.sh",
      "sudo systemctl restart app.service"
    ]
  }
}
```

**File provisioner use cases:**
- Upload configuration files to instances
- Deploy application code or binaries
- Provide certificates or credentials
- Transfer scripts that will be executed later
- Install custom modules or plugins

**Common connection types:**

1. **SSH** (for Linux):
```hcl
connection {
  type        = "ssh"
  host        = self.public_ip_address
  user        = "adminuser"
  
  # Authentication options (choose one):
  private_key = file("~/.ssh/id_rsa")                   # Private key file
  # password   = var.admin_password                      # Password authentication
  # agent      = true                                    # Use SSH agent
  # private_key_password = var.key_passphrase            # If key is password-protected
  
  # Advanced options
  port        = 22
  timeout     = "5m"
  script_path = "/tmp/terraform_%RAND%.sh"              # Path for uploaded scripts
}
```

2. **WinRM** (for Windows):
```hcl
connection {
  type     = "winrm"
  host     = self.public_ip_address
  user     = "adminuser"
  password = var.admin_password
  https    = true
  port     = 5986
  timeout  = "10m"
  
  # For self-signed certificates
  insecure = true
}
```

### Failure Behavior
By default, if a provisioner fails, the resource is marked as tainted. You can change this with `on_failure`:

```hcl
resource "azurerm_virtual_machine" "example" {
  # ... VM configuration ...

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
    
    on_failure = continue  # Options: continue or fail (default)
  }
}
```

### When to Use Provisioners
- **Best practice**: Prefer immutable infrastructure with pre-built images
- Use provisioners as a last resort when other methods aren't available
- Consider using dedicated configuration management tools like Ansible

### Creation-Time vs. Destroy-Time Provisioners
```hcl
resource "azurerm_resource_group" "example" {
  # ... resource group configuration ...

  # Runs when resource is created
  provisioner "local-exec" {
    command = "echo 'Resource group created: ${self.name}'"
  }

  # Runs when resource is destroyed
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Resource group being destroyed: ${self.name}'"
  }
}
```

## Functions and Expressions

### String Functions
```hcl
# String interpolation
name = "vm-${var.environment}-${random_string.suffix.result}"

# Format strings
name = format("vm-%s-%03d", var.environment, count.index + 1)

# String manipulation
title = title(var.environment)      # "Dev" => "Dev"
upper = upper(var.environment)      # "dev" => "DEV"
lower = lower(var.environment)      # "DEV" => "dev"
substr = substr(var.name, 0, 8)     # First 8 characters of name
```

### Numeric Functions
```hcl
max_vms = max(var.min_vms, var.desired_vms)
min_vms = min(var.max_vms, var.desired_vms)
ceiling = ceil(var.desired_capacity)  # Round up
floor = floor(var.desired_capacity)   # Round down
```

### Collection Functions
```hcl
# Merge maps
tags = merge(
  var.common_tags,
  {
    Name = "example-instance"
    Environment = var.environment
  }
)

# List operations
first_subnet = element(var.subnet_ids, 0)
contains_windows = contains(var.os_types, "windows")
distinct_zones = distinct(var.availability_zones)
length_of_list = length(var.subnet_ids)
```

### Type Conversion Functions
```hcl
to_string = tostring(var.port_number)
to_int = tonumber(var.string_port)
to_bool = tobool(var.enabled_string)
to_list = tolist(var.set_of_values)
to_map = tomap(var.tuple_of_key_value_pairs)
```

### Conditional Expressions
```hcl
# Ternary operator
vm_size = var.environment == "prod" ? "Standard_D2s_v3" : "Standard_B1s"

# Multiple conditions
vm_size = (
  var.environment == "prod" ? "Standard_D2s_v3" :
  var.environment == "staging" ? "Standard_B2s" :
  "Standard_B1s"
)
```

### For Expressions
```hcl
# Map transformation
environment_map = {
  for env in var.environments :
  env => upper(env)
}

# List transformation
vm_names = [
  for i in range(var.instance_count) :
  "vm-${var.environment}-${i + 1}"
]

# Filtered list
windows_vms = [
  for vm in var.virtual_machines :
  vm.name
  if vm.os_type == "windows"
]
```

### Splat Expressions
```hcl
# Get all subnet IDs
all_subnet_ids = azurerm_subnet.example[*].id

# Equivalent to:
all_subnet_ids = [
  for subnet in azurerm_subnet.example :
  subnet.id
]
```

## Meta-Arguments

### count
Creates multiple instances of a resource based on a number:

```hcl
resource "azurerm_subnet" "example" {
  count                = 3
  name                 = "subnet-${count.index}"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.${count.index}.0/24"]
}
```

### for_each
Creates multiple instances of a resource based on a map or set:

```hcl
# Using a map
resource "azurerm_network_security_rule" "example" {
  for_each                    = var.security_rules
  name                        = each.key
  priority                    = each.value.priority
  direction                   = each.value.direction
  access                      = each.value.access
  protocol                    = each.value.protocol
  source_port_range           = each.value.source_port_range
  destination_port_range      = each.value.destination_port_range
  source_address_prefix       = each.value.source_address_prefix
  destination_address_prefix  = each.value.destination_address_prefix
  resource_group_name         = azurerm_resource_group.example.name
  network_security_group_name = azurerm_network_security_group.example.name
}

# Using a set
resource "azurerm_resource_group" "example" {
  for_each = toset(["east", "west", "central"])
  name     = "rg-${each.key}"
  location = each.key == "east" ? "eastus" : each.key == "west" ? "westus" : "centralus"
}
```

### lifecycle
Controls resource lifecycle behavior:

```hcl
resource "azurerm_app_service" "example" {
  # ... configuration ...

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [
      tags,
      site_config[0].app_settings["LAST_MODIFIED"]
    ]
  }
}
```

### provider
Specifies which provider configuration to use:

```hcl
# Define multiple provider configurations
provider "azurerm" {
  features {}
  alias           = "subscription1"
  subscription_id = "subscription-id-1"
}

provider "azurerm" {
  features {}
  alias           = "subscription2"
  subscription_id = "subscription-id-2"
}

# Use a specific provider configuration
resource "azurerm_resource_group" "example" {
  provider = azurerm.subscription1
  name     = "example-resources"
  location = "East US"
}
```

### depends_on
Explicit dependencies (covered in Resource Dependencies section).

## Workspace Management

### What are Workspaces?
Workspaces allow you to manage multiple environments (e.g., dev, staging, prod) with the same configuration files but separate state files.

### Workspace Commands
```bash
# List workspaces
terraform workspace list

# Create a new workspace
terraform workspace new dev

# Select a workspace
terraform workspace select prod

# Show current workspace
terraform workspace show

# Delete a workspace
terraform workspace delete staging
```

### Using Workspaces in Configurations
```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-${terraform.workspace}"
  location = var.location
}

resource "azurerm_virtual_network" "example" {
  name                = "vnet-${terraform.workspace}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  
  # Define network properties based on workspace
  address_space = lookup({
    dev     = ["10.0.0.0/16"],
    staging = ["10.1.0.0/16"],
    prod    = ["10.2.0.0/16"]
  }, terraform.workspace)
}
```

### Workspace Best Practices
- Use for isolated testing environments
- Consider using different directories/configurations for production vs. non-production
- Avoid using workspaces for fundamentally different configurations

## State Operations

### Moving Resources
Rename or move resources without destroying and recreating them:

```bash
# Move resource to a different name
terraform state mv azurerm_virtual_network.old_name azurerm_virtual_network.new_name

# Move resource to a module
terraform state mv azurerm_subnet.public module.networking.azurerm_subnet.public
```

### Importing Existing Resources
Import resources that were created outside of Terraform:

```bash
# Import existing resource group
terraform import azurerm_resource_group.example /subscriptions/<subscription_id>/resourceGroups/existing-rg

# Import existing virtual network
terraform import azurerm_virtual_network.example /subscriptions/<subscription_id>/resourceGroups/existing-rg/providers/Microsoft.Network/virtualNetworks/existing-vnet
```

After importing, you need to write the corresponding configuration:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "existing-rg"
  location = "East US"
}

resource "azurerm_virtual_network" "example" {
  name                = "existing-vnet"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
}
```

### Refreshing State
Update state to match actual infrastructure:

```bash
terraform refresh
```

**Note**: This is now incorporated into `terraform plan` and `terraform apply` with the `-refresh-only` flag:

```bash
terraform apply -refresh-only
```

### State Manipulation
Advanced state operations:

```bash
# Remove resource from state without destroying it
terraform state rm azurerm_virtual_machine.example

# Pull state into a file
terraform state pull > terraform.tfstate

# Push state from a file
terraform state push terraform.tfstate
```

## Terraform Cloud and Enterprise Features

### Remote Backends
Terraform Cloud and Enterprise provide enhanced remote backend capabilities:

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "my-app-prod"
    }
  }
}
```

### Workspace-based Execution
Terraform Cloud/Enterprise provides:
- Remote execution environment
- Secure variable storage
- Team-based permissions
- Run triggers
- Cost estimation
- Policy checks (Sentinel)

### Private Module Registry
- Central location for sharing modules
- Version management
- Documentation
- Access controls

### Remote Operations
- Remote plan and apply
- Approval workflows
- Run history and logs
- VCS integration (GitHub, GitLab, etc.)

## Testing and Debugging

### Terraform Validate
Validates the syntax of your configuration:

```bash
terraform validate
```

### Terraform Console
Interactive console to test expressions:

```bash
terraform console
```

Example usage:
```
> var.environment
"dev"
> length(var.subnet_cidrs)
3
> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"
```

### Terraform Debug Logs
Set `TF_LOG` environment variable to see detailed logs:

```bash
export TF_LOG=DEBUG
terraform apply
```

Log levels: TRACE, DEBUG, INFO, WARN, ERROR

### Testing Modules
Tools for testing Terraform code:
- **Terratest**: Go framework for testing infrastructure
- **kitchen-terraform**: Integration with Test Kitchen
- **terraform-compliance**: BDD-style testing
- **checkov**: Static code analysis for security and compliance

### Common Debugging Steps
1. Use `terraform validate` to check syntax
2. Enable DEBUG logs to see API calls
3. Use `terraform console` to test expressions
4. Check resource dependencies
5. Examine the execution plan carefully

## Best Practices

### Project Structure
```
project/
├── main.tf              # Main configurations
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values (gitignored)
├── backend.tf           # Backend configuration
├── providers.tf         # Provider configurations
├── versions.tf          # Terraform and provider versions
├── modules/             # Local modules
│   └── networking/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/        # Environment-specific configs
    ├── dev/
    │   └── terraform.tfvars
    ├── staging/
    │   └── terraform.tfvars
    └── prod/
        └── terraform.tfvars
```

### Naming Conventions
```hcl
# Resource naming pattern
resource "azurerm_resource_group" "example" {
  name     = "${var.prefix}-${var.environment}-${var.location}-rg"
  location = var.location
}

# Tags for all resources
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project
    Owner       = var.owner
    ManagedBy   = "Terraform"
  }
}

resource "azurerm_virtual_network" "example" {
  name                = "${var.prefix}-${var.environment}-vnet"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
  
  tags = local.common_tags
}
```

### Version Constraints
```hcl
terraform {
  required_version = ">= 1.0.0, < 2.0.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"  # Allows 3.x but not 4.x
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.1.0"
    }
  }
}
```

### Security Best Practices
1. **Secure State Storage**:
   - Use remote backends with encryption
   - Restrict access to state files

2. **Secret Management**:
   - Use Azure Key Vault for sensitive values
   - Mark variables as sensitive
   - Don't store secrets in source control

3. **Least Privilege**:
   - Use service principals with minimum required permissions
   - Rotate credentials regularly

4. **Resource Protection**:
   - Use `prevent_destroy` for critical resources
   - Implement compliance policies

### Code Quality
1. **Consistent Formatting**:
   - Run `terraform fmt` before commits
   - Use a pre-commit hook

2. **Documentation**:
   - Document modules with README files
   - Add descriptions to variables and outputs
   - Use meaningful names

3. **Modularization**:
   - Break complex configurations into modules
   - Single responsibility principle

## Practical Examples

### Example 1: Basic Azure Infrastructure
```hcl
# Azure Resource Group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
  
  tags = {
    Environment = "Development"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
}

# Subnet
resource "azurerm_subnet" "example" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.2.0/24"]
}

# Network Interface
resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.example.id
    private_ip_address_allocation = "Dynamic"
  }
}

# Virtual Machine
resource "azurerm_linux_virtual_machine" "example" {
  name                = "example-machine"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_B1s"
  admin_username      = "adminuser"
  network_interface_ids = [
    azurerm_network_interface.example.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

### Example 2: Web Application with Database
```hcl
# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# App Service Plan
resource "azurerm_service_plan" "example" {
  name                = "example-appserviceplan"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  os_type             = "Linux"
  sku_name            = "B1"
}

# Web App
resource "azurerm_linux_web_app" "example" {
  name                = "example-webapp"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  service_plan_id     = azurerm_service_plan.example.id

  site_config {
    application_stack {
      node_version = "18-lts"
    }
  }

  app_settings = {
    "WEBSITE_NODE_DEFAULT_VERSION" = "~18"
    "DATABASE_URL"                = "postgresql://${azurerm_postgresql_server.example.fqdn}:5432/${azurerm_postgresql_database.example.name}"
  }
}

# PostgreSQL Server
resource "azurerm_postgresql_server" "example" {
  name                = "example-psqlserver"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  sku_name = "B_Gen5_1"

  storage_mb                   = 5120
  backup_retention_days        = 7
  geo_redundant_backup_enabled = false
  auto_grow_enabled            = true

  administrator_login          = "psqladmin"
  administrator_login_password = "H@Sh1CoRp2023!"
  version                      = "11"
  ssl_enforcement_enabled      = true
}

# PostgreSQL Database
resource "azurerm_postgresql_database" "example" {
  name                = "exampledb"
  resource_group_name = azurerm_resource_group.example.name
  server_name         = azurerm_postgresql_server.example.name
  charset             = "UTF8"
  collation           = "English_United States.1252"
}

# Firewall Rule (allow Azure services)
resource "azurerm_postgresql_firewall_rule" "example" {
  name                = "AllowAllAzureIPs"
  resource_group_name = azurerm_resource_group.example.name
  server_name         = azurerm_postgresql_server.example.name
  start_ip_address    = "0.0.0.0"
  end_ip_address      = "0.0.0.0"
}
```

### Example 3: Kubernetes Cluster with Container Registry
```hcl
# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# Container Registry
resource "azurerm_container_registry" "example" {
  name                = "exampleacr"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku                 = "Standard"
  admin_enabled       = false
}

# AKS Cluster
resource "azurerm_kubernetes_cluster" "example" {
  name                = "example-aks"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "exampleaks"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    Environment = "Development"
  }
}

# Role Assignment for AKS to access ACR
resource "azurerm_role_assignment" "example" {
  principal_id                     = azurerm_kubernetes_cluster.example.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.example.id
  skip_service_principal_aad_check = true
}

# Output kubeconfig
output "kube_config" {
  value     = azurerm_kubernetes_cluster.example.kube_config_raw
  sensitive = true
}

output "host" {
  value = azurerm_kubernetes_cluster.example.kube_config.0.host
}
```

### Example 4: Virtual Network Peering
```hcl
# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# Virtual Network 1
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet1"
  address_space       = ["10.1.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

# Virtual Network 2
resource "azurerm_virtual_network" "vnet2" {
  name                = "vnet2"
  address_space       = ["10.2.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

# Peering from vnet1 to vnet2
resource "azurerm_virtual_network_peering" "vnet1_to_vnet2" {
  name                      = "vnet1-to-vnet2"
  resource_group_name       = azurerm_resource_group.example.name
  virtual_network_name      = azurerm_virtual_network.vnet1.name
  remote_virtual_network_id = azurerm_virtual_network.vnet2.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
}

# Peering from vnet2 to vnet1
resource "azurerm_virtual_network_peering" "vnet2_to_vnet1" {
  name                      = "vnet2-to-vnet1"
  resource_group_name       = azurerm_resource_group.example.name
  virtual_network_name      = azurerm_virtual_network.vnet2.name
  remote_virtual_network_id = azurerm_virtual_network.vnet1.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
}
```

### Example 5: Azure Function App with Storage
```hcl
# Random string for unique names
resource "random_string" "random" {
  length  = 8
  special = false
  upper   = false
}

# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

# Storage Account
resource "azurerm_storage_account" "example" {
  name                     = "sa${random_string.random.result}"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

# App Service Plan
resource "azurerm_service_plan" "example" {
  name                = "example-appserviceplan"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  os_type             = "Windows"
  sku_name            = "Y1" # Consumption plan
}

# Function App
resource "azurerm_windows_function_app" "example" {
  name                = "func-${random_string.random.result}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  service_plan_id     = azurerm_service_plan.example.id

  storage_account_name       = azurerm_storage_account.example.name
  storage_account_access_key = azurerm_storage_account.example.primary_access_key

  site_config {
    application_stack {
      node_version = "~18"
    }
  }
}

# Function App Function
resource "azurerm_function_app_function" "example" {
  name            = "my-function"
  function_app_id = azurerm_windows_function_app.example.id
  language        = "JavaScript"
  
  file {
    name    = "index.js"
    content = <<EOF
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    
    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";
    
    context.res = {
        status: 200,
        body: responseMessage
    };
}
EOF
  }
  
  file {
    name    = "function.json"
    content = <<EOF
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
EOF
  }
}
```

---

This comprehensive crash course covers all essential aspects of Terraform for the Azure platform from the perspective of the Terraform Associate certification. Remember to consult the official HashiCorp documentation and Azure provider documentation for the most up-to-date information.

Happy Terraforming!

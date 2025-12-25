# Terraform Provider

## What is Provider?
* Provider is a plugin that is used to interact with the cloud resources.
## Types of Providers
* official providers  - These are provided by HashiCorp
  * azure
  * aws
  * gcp
* community providers   - These are provided by the community
* custom providers      - These are provided by the user

```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.57.0"
    }
  }
  required_version = ">= 1.14.0"
}

provider "azurerm" {
  features {}
}
```
#  Terraform Variables

This document explains **Terraform variables in depth**, covering:
- What variables are and why they exist
- Input variables, locals, and output variables
- How and when to use each
- Many real-world DevOps & Azure-style examples
- Clear mental models and best practices

---

## What Are Terraform Variables?

Terraform variables are **placeholders for values** that make infrastructure:
- Reusable
- Configurable
- Environment-aware
- Safe for automation

Terraform code without variables becomes **hardcoded and fragile**.

---
## Why Variables Are Required

### ❌ Without Variables
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "rg-dev-centralindia"
  location = "centralindia"
}
```

Problems:
- Cannot reuse for prod
- Cannot use CI/CD
- Must edit code every time

---

### ✅ With Variables
```hcl
resource "azurerm_resource_group" "rg" {
  name     = var.rg_name
  location = var.location
}
```

**Benefits:**
- Same code works everywhere
- Values come from outside
- Safer deployments

---

## Terraform Variable Categories (Big Picture)

Terraform variables are grouped into **three categories**:

1. Input Variables → values coming INTO Terraform
2. Local Values → computed values INSIDE Terraform
3. Output Variables → values going OUT of Terraform

### Mental Model
```text
Input Variables → Locals → Resources → Outputs
```
## 1. Input Variables (`variable`)

### 1.1 What Are Input Variables?

Input variables allow values to be **passed into Terraform** from:
- terraform.tfvars
- CLI arguments
- Environment variables
- CI/CD pipelines
- Parent modules

They behave like **function parameters**.

---

### 1.2 Basic Input Variable Example

```hcl
variable "location" {
  type        = string
  description = "Azure region"
}
```

**Usage:**
```hcl
location = var.location
```

---

### 1.3 Input Variable with Default Value

```hcl
variable "env" {
  type    = string
  default = "dev"
}
```

---
### 1.4 Input Variable Types (With Examples)
**String**
```hcl
variable "vm_size" {
  type = string
}
```
**Used for:**
* Names
* Regions
* SKUs

**Number**
```hcl
variable "vm_count" {
  type = number
}
```
**Used for:**
* VM count
* Disk size
* Node count

**Boolean**
```hcl
variable "enable_public_ip" {
  type = bool
}
```
**Used for:**
* Feature flags
* Enable/disable components

**List**
```hcl
variable "subnets" {
  type = list(string)
}
```

**Used for:**
* Subnets
* Availability zones

**Map**
```hcl
variable "tags" {
  type = map(string)
}
```

**Used for:**
* Tags
* Key-value configs

**Object**
```hcl
variable "vm_config" {
  type = object({
    size      = string
    disk_size = number
    public_ip = bool
  })
}
```

**Used for:**
* Grouping related values
* Module inputs

**Tags**
```hcl
variable "tags" {
  type = map(string)
}
```
**Used for:**
* Tags
* Key-value configs

**Object**
```hcl
variable "vm_config" {
  type = object({
    size      = string
    disk_size = number
    public_ip = bool
  })
}
```

**Used for:**
* Grouping related values
* Module inputs

### 1.5 Passing Values to Input Variables
**terraform.tfvars**
```hcl
location = "centralindia"
env      = "dev"
```

**CLI**
```terraform apply -var="env=prod"
```
**Environment Variable**
```tf
export TF_VAR_env=prod
```

## 2. Local Values (`locals`)
### 2.1 What Are Locals?
Locals are internal computed values used inside Terraform.
* They:
    * Are derived from input variables
    * Cannot be overridden
    * Exist only within the module
    * Improve readability and reuse
### 2.2 Why Locals Exist (Problem-Solution)
* ❌ Without Locals
```tf
name = "${var.project}-${var.env}-${var.region}-rg"
```

* Repeated everywhere → messy.

* ✅ With Locals
```hcl
locals {
  prefix = "${var.project}-${var.env}-${var.region}"
}
```

### 2.3 Naming Convention Example
```hcl
locals {
  base_name = "${var.project}-${var.env}-${var.region}"
}
```
**Usage:**

```hcl
name = "${local.base_name}-vnet"
```

### 2.4 Environment-Based Conditional Logic
```hcl
locals {
  disk_sku = var.env == "prod" ? "Premium_LRS" : "Standard_LRS"
}
```

**Result:**

* prod → Premium_LRS

* non-prod → Standard_LRS

### 2.5 Common Tags Example (Very Common)
```hcl
locals {
  common_tags = {
    Project     = var.project
    Environment = var.env
    Owner       = "DevOps"
    ManagedBy   = "Terraform"
  }
}
```

**Usage:**

* tags = local.common_tags

### 2.6 Feature Flags Using Locals
```hcl
locals {
  enable_monitoring = var.env == "prod"
}
```
**Usage:**
* monitoring_enabled = local.enable_monitoring

### 2.7 Data Transformation Example (Azure Storage)
```hcl
locals {
  sa_name = substr(
    lower(replace("${var.project}${var.env}${var.region}", "-", "")),
    0,
    24
  )
}
```

**Ensures:**
* Lowercase
* No hyphens
* Max length 24

### 2.8 Map Lookup Using Locals
```hcl
locals {
  region_code = {
    centralindia = "cin"
    eastus       = "eus"
  }
}
```
**Usage:**    
```hcl
name = "${var.project}-${local.region_code[var.region]}-rg"
```
### 2.9 When to Use Locals
**Use locals when:**
* Value is derived
* Used multiple times
* Must enforce logic
* Should not be overridden

### 2.10 When NOT to Use Locals
**When NOT to Use Locals:**
* When user controls value
* When environment overrides value
* When module caller decides

## 3. Output Variables (`output`)
### 3.1 What Are Outputs?
Outputs expose Terraform values after apply.
**They:**
* Show values to users
* Share values across modules
* Feed CI/CD pipelines

### 3.2 Basic Output Example
```hcl
output "rg_name" {
  value = azurerm_resource_group.rg.name
}
```

### 3.3 Module Output Example
```hcl
output "aks_cluster_id" {
  value = azurerm_kubernetes_cluster.aks.id
}
```
**Used in parent:**

```hcl
module.aks.aks_cluster_id
```

### 3.4 Output for CI/CD
**Common outputs:**
* Public IP
* DNS name
* Resource ID

### Variables vs Locals vs Outputs

| Feature	| Input	| Local	| Output	|
|-----------|-------|-------|---------|
| User controlled	| Yes	| No	| No |
| Computed	        | No	| Yes	| No |
| Internal only	    | No	| Yes	| No |
| Shared outside	| No	| No	| Yes |

### Best Practices
* Separate variables.tf, locals.tf, outputs.tf      
* Use locals for naming & tags
* Validate critical inputs
* Avoid secrets in variables
* Keep logic readable

### One-Line Rule to Remember
* Variables bring values in, locals clean them up, outputs send values out

---





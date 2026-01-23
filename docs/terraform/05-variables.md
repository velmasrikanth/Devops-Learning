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
### 1.4 Input Variable Data Types (With Examples)
* Generally Data types are divided into 2 categories
    * **Primitive** Data Types 
    * **Complex** Data Types
#### 1.4.1 Primitive Data Types
* These are single values
* Examples:
    * **String** - Textual values
    * **Number** - Numeric values
    * **Boolean** - True/False values

1. **String**
* A simple text value

* **Used for:**
  * Resource names
  * Regions
  * SKUs
  * Environment names

```hcl
variable "env" {
  type = string
  default = "dev"
  description = "Environment name"
}
variable "location" {
  type = string
  default = "centralindia"
  description = "Azure region"
}
```
```hcl
resource "azurerm_resource_group" "rg" {
  name = "${var.env}-rg"
  location = var.location
}
```

* **Key Points:**
  * Most Used type
  * Always Quoted (" ")
  * Used for Identifiers and Names 
  
2. **Number**
* A simple numeric value (integers or decimals)

* **Used for:**
  * VM count
  * Disk size
  * Node count

```hcl
variable "vm_count" {
  type = number
  default = 1
}
```
```hcl
resource "azurerm_virtual_machine" "vm" {
  count = var.vm_count
}
```

* **Key Points:**
  * Always Unquoted (1, 2, 3)
  * Terraform decides int vs float automatically

3. **Boolean**
* A simple true/false value

* **Used for:**
  * Feature flags
  * Enable/disable components

```hcl
variable "enable_public_ip" {
  type = bool
  default = true
}
```
```hcl
resource "azurerm_virtual_machine" "vm" {
  public_ip_address = var.enable_public_ip
}
```

* **Key Points:**
  * Always Unquoted (true, false) - If you put in quotes it will be string
  * Used for feature flags

#### 1.4.2 Complex Data Types
* These are multiple collection of values
* Examples:
    * **List** - **Ordered**, all elements must be of the **Same** type
    * **Set** - **Unordered**, all elements must be of the **Same** type (duplicates removed)
    * **Tuple** - **Ordered**, elements can be of **Different** type
    * **Map** - Key-value pairs
    * **Object** - Group of related values

* First we will see what is Ordered and Unordered and Nested
* 📚 **Ordered** vs **Unordered** vs **Nested**

1. **Ordered**
  - **Definition:** The sequence of elements matters. Terraform preserves the order you define.
  - **Implication:** You can access elements by their position (index).
  - **Types:** list, tuple.
  - **Usage:** When order matters (e.g., priority, sequence).

```hcl
variable "ports" {
  type    = list(number)
  default = [22, 80, 443]
}
# ports[0] = 22, ports[1] = 80, ports[2] = 443
```
* 👉 If you loop over this list, the rules will be created in the same order. Useful when priority or sequence matters.

2. **Unordered**
  - **Definition:** The sequence of elements doesn't matter. Terraform doesn't preserve the order.
  - **Implication:** You can't access elements by position (index).
  - **Types:** `set`, `map`, `object`.
```hcl
variable "subnets" {
  type    = set(string)
  default = ["10.0.0.0/24", "10.0.1.0/24", "10.0.2.0/24"]
}

# subnet1, subnet2, subnet3 (order not guaranteed)
```
  
* 👉 If you loop over this set, the order might be different each time. Useful when order doesn't matter.
* 👉 Terraform doesn’t care about order here. You just know the set contains {10.0.0.0/24, 10.0.1.0/24, 10.0.2.0/24}. Perfect when uniqueness is more important than order.

3. **Nested**
  - **Definition:** - Nested means combining one complex type inside another (like a hierarchy).
  - **Implication:**  Lets you model real‑world structures (e.g., AKS node pools, VM configs).
  - **Types:**  Any complex type can be nested (e.g.,`map(object(...))`, `list(object(...))`)

```hcl
variable "node_pools" {
  type = map(object({
    vm_size            = string
    node_count         = number
    enable_auto_scaling = bool
  }))
  default = {
    system = {
      vm_size            = "Standard_DS2_v2"
      node_count         = 2
      enable_auto_scaling = false
    }
    user = {
      vm_size            = "Standard_DS3_v2"
      node_count         = 3
      enable_auto_scaling = true
    }
  }
}

resource "azurerm_kubernetes_cluster_node_pool" "pool" {
    for_each            = var.node_pools
    name                = each.key
    vm_size             = each.value.vm_size
    node_count          = each.value.node_count
    enable_auto_scaling = each.value.enable_auto_scaling
}   
```   
* 👉 Here, you have a map (unordered keys: string, values: object) containing objects (structured configs). That’s nesting.

* Now let's see about Complex Data Types    

1. **List (Ordered,Same Type)**
* Ordered collection of values of the same type.
* **Used For:**
    * Defining NSG inbound ports with priority.
    * Listing availability zones for VMs.
```hcl
variable "allowed_ports" {
  type    = list(number)
  default = [22, 80, 443]
}
```
* **Key Points:**
    * Preserves order.
    * Access elements by index (e.g., list[0], list[1]).
    * Useful when order matters (e.g., priority, sequence).

2. **Set (Unordered,Same Type)**
* Unordered collection of values of the same type.
* **Used For:**
    * Defining NSG inbound ports with priority.
    * Listing availability zones for VMs.
```hcl
variable "unique_ports" {
  type    = set(number)
  default = [22, 22, 443] # becomes {22, 443}
}

```
* **Key Points:**
    * Unordered collection.
    * No duplicates allowed.
    * Useful when order doesn't matter (e.g., unique values).

3. **Tuple (Ordered, Different Types)**
* Ordered collection of values of different types.
* **Used For:**
    * Defining VM configuration (size, count, auto-scaling).
    * Representing fixed resource settings like SKU, tier, and flags.

```hcl
variable "vm_config" {
  type    = tuple([string, number, bool])
  default = ["Standard_DS1_v2", 2, true]
}

```
* **Key Points:**
    * Preserves order.
    * Access elements by index (e.g., tuple[0], tuple[1]).
    * Useful when order matters (e.g., priority, sequence).

4. **Map (Unordered, Key-Value, Same Type)**
* Unordered key-value pairs with same-type values.
* **Used For:**
    * Defining resource tags (environment, owner).
    * Mapping region-to-location codes.
```hcl
variable "tags" {
  type = map(string)
  default = {
    environment = "dev"
    owner       = "srikanth"
  }
}

```
* **Key Points:**
    * Keys are strings.
    * Values must be same type.

5. **Object (Ordered, Key-Value, Mixed Type)**
* Ordered collection of key-value pairs.
* **Used For:**
    * Defining NSG inbound ports with priority.
    * Listing availability zones for VMs.
```hcl
variable "allowed_ports" {
  type    = object({
    port    = number
    protocol = string
  })
  default = {22 = "tcp", 80 = "tcp", 443 = "tcp"}
}
```
* **Key Points:**
    * Preserves order.
    * Access elements by index (e.g., object[0], object[1]).
    * Useful when order matters (e.g., priority, sequence).

### 1.5 Passing Values to Input Variables
**terraform.tfvars**
```hcl
location = "centralindia"
env      = "dev"
```

**CLI**
```bash
terraform apply -var="env=prod"
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





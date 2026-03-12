---
name: terraform
description: |
  Terraform expert for multi-cloud infrastructure as code.
  Generates modules, state management configs, provider setups, and CI/CD integration.
  Works across AWS, Azure, GCP, Databricks, Snowflake, and other providers.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[task: module|state|import|migrate|review] [cloud/resource description]"
---

## Role

You are a Terraform specialist. You write infrastructure code that is modular, testable, and safe to apply in production.

## Task Router

| Task | What You Do |
|------|------------|
| `module` | Generate a reusable Terraform module for a described resource/pattern |
| `state` | Configure remote state backends, workspaces, state management |
| `import` | Generate import blocks for existing infrastructure |
| `migrate` | Migrate between state backends, refactor modules, move resources |
| `review` | Review existing Terraform code for best practices, security, DRY |

---

## Project Structure

### Standard Layout

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf          # Module calls with dev values
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf       # Dev state backend
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── compute/
│   ├── database/
│   └── monitoring/
└── shared/
    ├── providers.tf          # Shared provider config
    └── versions.tf           # Required provider versions
```

### Module Template

```hcl
# modules/<name>/main.tf

terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# ── Resources ────────────────────────────────

resource "aws_example" "this" {
  name = var.name
  # ...

  tags = merge(var.common_tags, {
    Name = var.name
  })
}

# modules/<name>/variables.tf

variable "name" {
  description = "Resource name"
  type        = string

  validation {
    condition     = length(var.name) > 0 && length(var.name) <= 64
    error_message = "Name must be 1-64 characters."
  }
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "common_tags" {
  description = "Tags applied to all resources"
  type        = map(string)
  default     = {}
}

# modules/<name>/outputs.tf

output "id" {
  description = "Resource ID"
  value       = aws_example.this.id
}

output "arn" {
  description = "Resource ARN"
  value       = aws_example.this.arn
}
```

---

## State Management

### Remote Backend Configs

#### AWS S3

```hcl
terraform {
  backend "s3" {
    bucket         = "myorg-terraform-state"
    key            = "environments/prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
    kms_key_id     = "alias/terraform-state"
  }
}
```

#### Azure Storage

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate"
    container_name       = "tfstate"
    key                  = "environments/prod/terraform.tfstate"
    use_oidc             = true  # Prefer over access keys
  }
}
```

#### GCP GCS

```hcl
terraform {
  backend "gcs" {
    bucket = "myorg-terraform-state"
    prefix = "environments/prod"
  }
}
```

### State Rules
- **One state per environment per component** — never share state across envs
- **Always use remote state** — never local for team projects
- **Enable state locking** (DynamoDB/built-in)
- **Enable encryption** at rest
- **Use `terraform_remote_state` data source** sparingly — prefer SSM/Vault for cross-stack values

---

## Coding Conventions

### Naming

```hcl
# Resources: snake_case, descriptive
resource "aws_security_group" "web_app" { ... }

# Variables: snake_case, include unit for numerics
variable "max_retention_days" { ... }
variable "instance_count" { ... }

# Outputs: snake_case, prefixed with resource type
output "vpc_id" { ... }
output "alb_dns_name" { ... }

# Locals: computed values, prefixed for grouping
locals {
  name_prefix = "${var.project}-${var.environment}"
  common_tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "terraform"
  }
}
```

### Variables Best Practices

```hcl
# ✅ Always have: description, type, validation
variable "cidr_block" {
  description = "VPC CIDR block"
  type        = string
  
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block."
  }
}

# ✅ Use object types for complex inputs
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    storage_gb     = number
    multi_az       = bool
  })
}

# ✅ Sensitive values
variable "db_password" {
  description = "Database master password"
  type        = string
  sensitive   = true
}
```

### Resource Patterns

```hcl
# ✅ Use for_each over count (stable addressing)
resource "aws_subnet" "private" {
  for_each = var.private_subnets

  vpc_id            = aws_vpc.this.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-private-${each.key}"
    Tier = "private"
  })
}

# ✅ Use moved blocks for refactoring
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}

# ✅ Use import blocks (Terraform 1.5+)
import {
  to = aws_s3_bucket.existing
  id = "my-existing-bucket"
}

# ✅ Use lifecycle rules deliberately
resource "aws_db_instance" "this" {
  # ...
  lifecycle {
    prevent_destroy = true           # Safety for databases
    ignore_changes  = [password]     # Managed externally
  }
}
```

---

## Provider-Specific Patterns

### Databricks Provider

```hcl
terraform {
  required_providers {
    databricks = {
      source  = "databricks/databricks"
      version = "~> 1.0"
    }
  }
}

# Unity Catalog
resource "databricks_catalog" "analytics" {
  name    = "analytics"
  comment = "Analytics catalog for gold-layer data"
}

resource "databricks_schema" "sales" {
  catalog_name = databricks_catalog.analytics.name
  name         = "sales"
  comment      = "Sales domain tables"
}

# Cluster Policy
resource "databricks_cluster_policy" "data_engineering" {
  name = "Data Engineering Policy"
  definition = jsonencode({
    "autotermination_minutes" : { "type" : "fixed", "value" : 60 },
    "spark_version" : { "type" : "regex", "pattern" : "14\\.[0-9]+\\.x-scala.*" },
    "node_type_id" : { "type" : "allowlist", "values" : [
      "Standard_D4ds_v5", "Standard_D8ds_v5"
    ]}
  })
}
```

### Snowflake Provider

```hcl
terraform {
  required_providers {
    snowflake = {
      source  = "Snowflake-Labs/snowflake"
      version = "~> 0.90"
    }
  }
}

resource "snowflake_database" "analytics" {
  name                        = "ANALYTICS"
  data_retention_time_in_days = 30
  comment                     = "Business-ready analytics"
}

resource "snowflake_warehouse" "etl" {
  name                = "ETL_WH"
  warehouse_size      = "XSMALL"
  auto_suspend        = 60
  auto_resume         = true
  min_cluster_count   = 1
  max_cluster_count   = 1
  scaling_policy      = "STANDARD"
  resource_monitor    = snowflake_resource_monitor.etl.name
}
```

---

## Safety & CI/CD

### Pre-Apply Checklist
- [ ] `terraform fmt -check` passes
- [ ] `terraform validate` passes
- [ ] `terraform plan` reviewed — no unexpected destroys
- [ ] No secrets in `.tf` files or `.tfvars` committed to git
- [ ] State backend is configured (not local)
- [ ] Provider versions pinned with `~>` constraint

### CI Pipeline Stages

```
fmt -check → validate → plan → (manual approve) → apply
```

### Dangerous Operations (require explicit confirmation)
- Any `destroy` action
- Replacing resources that cause downtime (DB, stateful services)
- Changing `backend` configuration
- Removing `prevent_destroy` lifecycle rules

---

## Review Checklist

| Area | Check |
|------|-------|
| **Structure** | Modules for reusable components? Env separation? |
| **State** | Remote backend? Locking? Encryption? Separate per env? |
| **Variables** | Descriptions? Types? Validation? Sensitive marked? |
| **Resources** | `for_each` over `count`? Tags on everything? Lifecycle rules on stateful? |
| **Security** | No secrets in code? Least-privilege IAM? Encryption enabled? |
| **Versions** | Providers pinned? Terraform version constrained? |
| **Outputs** | Useful outputs for downstream consumers? |
| **DRY** | Repeated blocks extracted to modules? Locals for computed values? |

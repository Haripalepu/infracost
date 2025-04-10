# Terraform Module for AWS Aurora PostgreSQL

This module provisions an Amazon Aurora PostgreSQL cluster with support for optional role creation, database initialization, encrypted password handling using AWS KMS, and automated credential storage in AWS Secrets Manager.

---

## Use Cases

- Provision a secure and automated Amazon Aurora PostgreSQL cluster using Terraform
- Create databases and user roles within the Aurora cluster during provisioning
- Automatically generate passwords, encrypt them using AWS KMS, and store them in AWS Secrets Manager
- Integrate with existing VPC, Subnets, and Security Groups for network configuration
- Customize PostgreSQL behavior using parameter groups and additional cluster-level settings
- Ensure code consistency by running `terraform fmt` from the current working directory before committing changes
- Upon creating a Pull Request, Atlantis will run `terraform plan` and post the output in the PR. Verify that only the intended changes are present
- All changes should be reviewed and approved by the Platform team to ensure system integrity before applying
- Once approved, comment `atlantis apply` on the PR to apply the changes


---

## Getting Started

To use this module, you need access to a valid AWS account and a role such as `KlaraProductDeveloperAccess` with required permissions to manage RDS, Secrets Manager, and KMS resources.

Before you begin:
1. Ensure you have access to the `terraform/password-encryption` KMS key.
2. Use the `encrypt_tool.sh` to encrypt custom passwords if needed, or let the module generate passwords automatically.
3. Add the password (encrypted) to your environment-specific `.tfvars` file or use the auto-generate option by leaving it blank.

---

## Example Configuration

```hcl
module "aurora_postgresql" {
  source  = "git::https://github.com/your-org/terraform-aws-aurora-postgresql.git"

  name                      = "aurora-stg"
  vpc_id                   = var.vpc_id
  subnet_ids               = var.subnet_ids
  kms_key_id               = var.kms_key_id
  engine_version           = "15.3"
  instance_class           = "db.r6g.large"
  database_name            = "ehr_integration"
  master_username          = "admin"
  master_password_secret_arn = var.master_password_secret_arn
  apply_immediately        = true

  roles = {
    casper = {
      password = "<encrypted-password>" # Optional
    }
  }

  tags = {
    Product     = "ehr"
    Environment = "staging"
  }
}
```

---

## Roles Example

```hcl
roles = {
  casper = {
    password = "<encrypted-string>"
  },
  analytics = {}
}
```

If `password` is not provided, a random password is created and stored securely in AWS Secrets Manager.

---

## Databases Example

```hcl
databases = {
  ehr_integration = {
    extensions  = ["uuid-ossp", "pg_stat_statements"]
    privileges  = ["CONNECT", "TEMPORARY"]
  }
}
```

---
<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.5.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 4.0 |
| <a name="requirement_random"></a> [random](#requirement\_random) | ~> 3.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 4.0 |
| <a name="provider_random"></a> [random](#provider\_random) | ~> 3.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_admin_credential_secret"></a> [admin\_credential\_secret](#module\_admin\_credential\_secret) | git::ssh://git@github.com/ModMedPD/terraform-aws-secret-manager | 1.1.2 |
| <a name="module_aurora_postgres"></a> [aurora\_postgres](#module\_aurora\_postgres) | terraform-aws-modules/rds-aurora/aws | ~> 9.0 |
| <a name="module_aurora_postgres_configuration"></a> [aurora\_postgres\_configuration](#module\_aurora\_postgres\_configuration) | ./modules/postgresql-configuration | n/a |
| <a name="module_aurora_postgres_security_group"></a> [aurora\_postgres\_security\_group](#module\_aurora\_postgres\_security\_group) | terraform-aws-modules/security-group/aws | ~> 5.0 |
| <a name="module_kms"></a> [kms](#module\_kms) | git::ssh://git@github.com/ModMedPD/terraform-aws-kms | 1.2.0 |
| <a name="module_role_credential_secret"></a> [role\_credential\_secret](#module\_role\_credential\_secret) | git::ssh://git@github.com/ModMedPD/terraform-aws-secret-manager | 1.1.2 |

## Resources

| Name | Type |
|------|------|
| [aws_caller_identity.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) | data source |
| [aws_default_tags.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/default_tags) | data source |
| [aws_iam_policy_document.kms](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_kms_secrets.admin_credential](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/kms_secrets) | data source |
| [aws_kms_secrets.role_credentials](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/kms_secrets) | data source |
| [aws_region.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |
| [aws_security_groups.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/security_groups) | data source |
| [aws_subnets.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/subnets) | data source |
| [aws_vpc.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/vpc) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_configuration_enabled"></a> [configuration\_enabled](#input\_configuration\_enabled) | Bool to decide if one wants to configure Aurora PostgreSQL RDS instance with DBs,roles,permissions | `bool` | `true` | no |
| <a name="input_databases"></a> [databases](#input\_databases) | Database names created inside Aurora PostgreSQL instance | `any` | n/a | yes |
| <a name="input_enabled"></a> [enabled](#input\_enabled) | Bool to decide if RDS resources are created | `bool` | `true` | no |
| <a name="input_instances"></a> [instances](#input\_instances) | n/a | `map(object({}))` | <pre>{<br/>  "one": {},<br/>  "three": {},<br/>  "two": {}<br/>}</pre> | no |
| <a name="input_name"></a> [name](#input\_name) | Name of Aurora PostgreSQL instance | `string` | n/a | yes |
| <a name="input_roles"></a> [roles](#input\_roles) | Roles created inside of Aurora PostgreSQL instance | <pre>map(object({<br/>    password  = optional(string)<br/>    member_of = optional(list(string), [])<br/>  }))</pre> | `{}` | no |
| <a name="input_settings"></a> [settings](#input\_settings) | Settings of Aurora PostgreSQL instance | <pre>object({<br/>    admin_password                    = optional(string)<br/>    engine_version                    = optional(string, "15.3")<br/>    engine_mode                       = optional(string, "provisioned")<br/>    apply_immediately                 = optional(bool, true)<br/>    multi_az                          = optional(bool, false)<br/>    vpc_name                          = optional(string) # VPC Name ex klara/stg - if not provided name is generated from tags $product/$environment<br/>    subnet_type                       = optional(string, "database")<br/>    storage_type                      = optional(string, "gp3") # Storage type gp2,gp3,io1,io2 https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Storage.html#gp3-storage<br/>    iops                              = optional(number)        # Storage size lower than 400GB can't set iops; for io storage type minimum value is 1000<br/>    throughput                        = optional(number)        # Storage size lower than 400GB can't set throughput<br/>    allocated_storage                 = optional(number, 20)    # Minimal size for GP3 is 20G<br/>    backup_window                     = optional(string, "04:00-05:00")<br/>    backup_retention_days             = optional(number, 7) # Snapshot retention in days<br/>    maintenance_window                = optional(string, "Mon:07:00-Mon:09:00")<br/>    source_cidrs                      = list(string)               # Allowed CIDR blocks - used in security group<br/>    source_security_group_names       = optional(list(string), []) # Allowed Source Security Group Names; Used as filter to query Security Group ID<br/>    log_retention_days                = optional(number, 7)        # CloudWatch log retention in days<br/>    deletion_protection               = optional(bool, true)<br/>    min_capacity                      = optional(number, 0.5)<br/>    max_capacity                      = optional(number, 2.0)<br/>    autoscaling_enabled               = optional(bool, false)<br/>    db_cluster_parameter_group_family = optional(string, "aurora-postgresql15")<br/>    instance_class                    = optional(string, "db.r6g.large")<br/><br/>    parameters = optional(map(object({<br/>      value        = string<br/>      apply_method = optional(string, "immediate")<br/>    })))<br/>  })</pre> | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_endpoint"></a> [endpoint](#output\_endpoint) | Aurora RDS PostreSQL instance ID |
<!-- END_TF_DOCS -->
## Notes

- Passwords generated automatically are stored under:
  ```
  rds/cluster-name/role-name/db-name
  ```
- Aurora clusters are accessible only within VPC.
- Use a jump box, bastion host, or VPN (e.g. Client VPN or EKS pod with psql) to access them.

---



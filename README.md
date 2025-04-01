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

## Terraform Requirements

| Name       | Version |
|------------|---------|
| terraform  | >= 1.5.0 |
| aws        | >= 4.0   |
| random     | ~> 3.0   |

---

## Terraform Providers

| Name   | Version |
|--------|---------|
| aws    | >= 4.0  |
| random | ~> 3.0  |

---

## Modules Used

| Name                    | Source                                                   | Version |
|-------------------------|----------------------------------------------------------|---------|
| secret_manager          | git::https://github.com/ModMedPD/terraform-aws-secret-manager | 1.1.2  |
| kms                     | git::https://github.com/ModMedPD/terraform-aws-kms       | 1.0.5  |
| postgres_configuration  | ./modules/postgresql-configuration                       | n/a     |
| security_group          | terraform-aws-modules/security-group/aws                 | ~> 5.0  |

---

## Resources Created

### Resources Created

| Name                                                                                                 | Type      |
|------------------------------------------------------------------------------------------------------|-----------|
| [`aws_rds_cluster`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster) | Resource |
| [`aws_rds_cluster_instance`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/rds_cluster_instance) | Resource |
| [`aws_security_group`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | Resource |
| [`aws_secretsmanager_secret`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret) | Resource |
| [`aws_secretsmanager_secret_version`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret_version) | Resource |
| [`aws_db_subnet_group`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_subnet_group) | Resource |


---

## Inputs

| Name                        | Description                                                    | Type   | Default | Required |
|-----------------------------|----------------------------------------------------------------|--------|---------|----------|
| name                        | Cluster name                                                   | string | n/a     | yes      |
| vpc_id                      | VPC ID                                                         | string | n/a     | yes      |
| subnet_ids                  | Subnet IDs for DB Subnet Group                                 | list   | n/a     | yes      |
| kms_key_id                  | KMS Key ID for encryption                                      | string | n/a     | yes      |
| engine_version              | Aurora PostgreSQL version                                      | string | "15.3"  | no       |
| instance_class              | DB instance type                                               | string | "db.r6g.large" | no  |
| database_name               | Default database name                                          | string | ""      | no       |
| master_username             | DB admin username                                              | string | n/a     | yes      |
| master_password_secret_arn  | Secret ARN for master password                                | string | n/a     | yes      |
| apply_immediately           | Whether to apply changes immediately                          | bool   | false   | no       |
| roles                       | DB user roles with optional password/member_of                | map    | {}      | no       |
| databases                   | Databases to create with optional extensions and privileges   | map    | {}      | no       |
| tags                        | Additional resource tags                                       | map    | {}      | no       |

---

## Outputs

| Name               | Description                                      |
|--------------------|--------------------------------------------------|
| cluster_endpoint   | Endpoint of the writer instance                  |
| reader_endpoint    | Reader endpoint of the Aurora cluster            |
| cluster_arn        | ARN of the Aurora cluster                        |

---

## Notes

- Passwords generated automatically are stored under:
  ```
  rds/cluster-name/role-name/db-name
  ```
- Aurora clusters are accessible only within VPC.
- Use a jump box, bastion host, or VPN (e.g. Client VPN or EKS pod with psql) to access them.

---


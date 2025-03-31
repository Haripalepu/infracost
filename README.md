# terraform-aws-aurora-postgresql

Terraform module to provision an Amazon Aurora PostgreSQL cluster in AWS with recommended settings including encryption, parameter group, subnet group, and security groups.

## Features

- Creates an Aurora PostgreSQL cluster
- Supports multi-AZ deployment
- Integrates with AWS Secrets Manager
- Uses KMS for encryption
- Configurable DB parameters
- Configurable security groups and ingress rules
- Option to enable monitoring (e.g., Datadog)

## Usage

```hcl
module "aurora_postgresql" {
  source  = "git::https://your-repo-url/terraform-aws-aurora-postgresql.git"

  cluster_identifier       = "my-aurora-cluster"
  engine_version           = "15.3"
  instance_class           = "db.r6g.large"
  subnet_ids               = var.subnet_ids
  vpc_id                   = var.vpc_id
  kms_key_id               = var.kms_key_id
  database_name            = "mydb"
  master_username          = "admin"
  master_password_secret_arn = var.master_password_secret_arn
  apply_immediately        = true

  tags = {
    Environment = "dev"
    Project     = "aurora"
  }
}
```

## Inputs

| Name                        | Description                                          | Type   | Default | Required |
|-----------------------------|------------------------------------------------------|--------|---------|----------|
| `cluster_identifier`        | Unique name for the DB cluster                       | string | n/a     | yes      |
| `engine_version`            | Aurora PostgreSQL engine version                     | string | n/a     | yes      |
| `instance_class`            | DB instance class                                    | string | n/a     | yes      |
| `subnet_ids`                | List of subnet IDs for DB subnet group               | list   | n/a     | yes      |
| `vpc_id`                    | VPC ID for security group creation                   | string | n/a     | yes      |
| `kms_key_id`                | KMS key for encryption                               | string | n/a     | yes      |
| `database_name`             | Name of the default database to create               | string | `""`    | no       |
| `master_username`           | Master username for the cluster                      | string | n/a     | yes      |
| `master_password_secret_arn` | ARN of secret storing the master password          | string | n/a     | yes      |
| `apply_immediately`         | Whether changes should be applied immediately        | bool   | false   | no       |
| `tags`                      | Tags to apply to resources                           | map    | `{}`    | no       |

## Outputs

| Name               | Description                             |
|--------------------|-----------------------------------------|
| `cluster_endpoint` | Writer endpoint of the DB cluster       |
| `reader_endpoint`  | Reader endpoint of the DB cluster       |
| `cluster_arn`      | ARN of the Aurora DB cluster            |

## Folder Structure

- `main.tf` – Core logic to create the Aurora cluster
- `kms.tf` – KMS integration for encryption
- `secrets.tf` – Integration with AWS Secrets Manager
- `security-groups.tf` – Security group rules
- `outputs.tf` – Outputs for easy integration
- `variables.tf` – Input variable definitions
- `modules/postgresql-configuration` – Optional configurations like Datadog

## Prerequisites

- Terraform 1.3 or higher
- AWS provider
- IAM permissions to create Aurora resources, Secrets Manager, KMS, and VPC components

## License

This module is licensed under the MIT License.

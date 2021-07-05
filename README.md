# Terraform AWS RDS Aurora

## About:

This module can deploy only AWS RDS Aurora clusters for Aurora, Postgres, and MySQL database engines. This can optionally include enhanced monitoring, and an auto scaling configuration.

### Resources deployed

This module deploys these resources;

- **``aws_db_subnet_group``**: At least 2 subnets on two different Availability Zones need to be assigned to this group via ``var.subnets`` variable.
- **``aws_rds_cluster``**: Aurora cluster configuration
- **``aws_rds_cluster_instance``**: Instance(s) within the Aurora cluster
- **``aws_db_parameter_group``**: Parameters that apply to the databases within the cluster.
- **``aws_rds_cluster_parameter_group``**: Parameters specifically for the cluster.
- (optional) Only deployed if enhanced monitoring is enabled. The ``var.monitoring_interval`` variable needs to have a number larger than ``0``
  - **``aws_iam_role``**
  - **``aws_iam_role_policy_attachment``**
- (optional) Only deployed if autoscaling is enabled via ``var.replica_scale_enabled`` set to ``true``. Additionally, ``var.replica_scale_min`` and ``var.replica_scale_max`` needs to be set or defaults are used.
  - **``aws_appautoscaling_target``**
  - **``aws_appautoscaling_policy``**


## How to use:


```hcl
module "aurora_mysql" {
  source = "github.com/eezze/terraform-aws-rds-aurora?ref=v1.0"

  name              = "${local.name}-mysql"
  engine            = "aurora-mysql"
  engine_mode       = "serverless"
  storage_encrypted = true

  vpc_id                = module.vpc.vpc_id
  subnets               = module.vpc.database_subnets

  replica_scale_enabled = false
  replica_count         = 0

  monitoring_interval = 60

  apply_immediately   = true
  skip_final_snapshot = true

  scaling_configuration = {
    auto_pause               = true
    min_capacity             = 2
    max_capacity             = 16
    seconds_until_auto_pause = 300
    timeout_action           = "ForceApplyCapacityChange"
  }

  db_parameter_group_parameters = [
    {
      name  = "aurora_lab_mode"
      value = 1
    },
    {
      name  = "optimizer_switch"
      value = "hash_join=on"
    }
  ]

  db_cluster_parameter_group_parameters = [
    {
      name  = "tls_version"
      value = "TLSv1.2"
    },
    {
      name  = "character_set_client"
      value = "utf8mb4"
    },
    {
      name  = "character_set_server"
      value = "utf8mb4"
    }
  ]
}

```

## Changelog

### v1.0
 - Initial release
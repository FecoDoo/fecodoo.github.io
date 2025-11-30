---
title: "Setting Up Docker Registry on AWS ECS"
date: 2023-06-01T00:49:36+03:00
draft: false
---

## Why do you need a Docker Pull-through Registry?

DockerHub, Docker's public registry, is a treasure trove of pre-built images that simplifies the deployment process for DevOps teams worldwide. However, as your containerized applications scale, you may find yourself hitting the limits of DockerHub's **fair-use policy**. This is where setting up your private Docker Pull-through Registry with **AWS Elastic Container Service (ECS)** comes to the rescue.  The `docker pull-through registry` allows for large-quantity and repeated image pulls, as well as speeding up pulling operations by aggressively caching images that may be required from multiple nodes in a Kubernetes cluster.

In this article, I will introduce you to the basic steps of creating a private Docker Registry with **AWS ECS** and **Terraform**.

<details>
  <summary>What benefits can you get?</summary>

- Reliability: Public Docker registries like DockerHub are indispensable during the development phase, but when it comes to production, reliability is key. By setting up your private Docker Registry, you gain control over your container images' availability, ensuring they are accessible when you need them. No more worrying about service interruptions or slowdowns due to high demand on DockerHub.

- Enhanced Security: Security is a top priority for any DevOps team. Private Docker Registries provide an additional layer of security by allowing you to control who can access and download your container images. You can implement authentication and authorization mechanisms to safeguard your intellectual property and sensitive data.

- Optimized Bandwidth Usage: DockerHub's fair-use policy limits the number of pulls you can perform, potentially disrupting your CI/CD pipelines. With a private Docker Registry, you can reduce your reliance on external registries, optimizing bandwidth usage, and ensuring consistent and faster downloads.

- Customization: Your container images may have specific requirements or configurations that are not readily available in public registries. By hosting your private Docker Registry, you have the flexibility to customize images to meet your application's unique needs.

- Compliance and Governance: Private Docker Registries are essential for industries with stringent compliance requirements, such as healthcare or finance. You can maintain control over your container images, ensuring they meet regulatory standards and internal governance policies.

- Reduced Latency: For global deployments, accessing container images from a private registry hosted in a region closer to your ECS cluster can significantly reduce latency, leading to faster application start times and improved user experience.

</details>

## Infrastructure

### Cloud Providers

To enable public access to the registry, the first step is to choose a cloud provider. In this tutorial, I will be using AWS as the cloud provider. Keep in mind that the configuration for other platforms may vary slightly, but you will still grasp the basic idea of how to create all the necessary resources.

### Components

For your convinience, all major components are listed here:

- `VPC`: Definition of the network structure with public/private subnets and `NAT gateways`
- `ECS`: Service which enalbes running registry containers as a service
- `S3`: **Persistant Storage** of keeping cached image manifests, layers, and also `ELB` logs
- `IAM`: Definition of `IAM roles` (with permissions) for accessing various `AWS` resources
- `ELB`: `Load Balancer` for accessing the registry from the public network
- `SG`: `Security Groups` with `inbound/outbound` rules
- `ACM`: Amazon managed certs
- `CloudWatch`: Metrics monitoring and service alerts
- `Secrets`: Various secrets

> Some components may import dynamic values (starts with **var.**). These values can be defined with tarraform `variable`.

### VPC

For the simplest use case, you will need a network with at least two subnets: one marked as public and another marked as private. The public subnet should be used exclusively for deploying NAT gateways, ELB, and any other services that face the public internet, in order to adhere to the best security practices. Services like ECS tasks should only be deployed in the private subnet.

```terraform
# Create a VPC
resource "aws_vpc" "vpc" {
  cidr_block = ["10.0.0.0"] # CIDR mask for subnets seperation
}
```

#### Subnets

```json
# Here we define two subnets:

# Public subnet
resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.vpc.id
  availability_zone = var.azs
  cidr_block        = ["10.0.0.0/16"]
}

# Private subnet
resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.vpc.id
  availability_zone = var.azs
  cidr_block        = ["10.1.0.0/16"]
}

```


#### Gateways, routes and IPs

```json
# Create elastic ips which can be later assigned to gateways
resource "aws_eip" "eip" {
  vpc = true
}

# Associate Route Table with internet gateway and public nat gateway
resource "aws_route_table" "igw_route_table" {
  vpc_id = aws_vpc.vpc.id
}

# Create IGW route
resource "aws_route" "igw_route" {
  gateway_id             = aws_internet_gateway.gateway.id
  route_table_id         = aws_route_table.igw_route_table.id
  destination_cidr_block = "0.0.0.0/0"
}

# Create IGW route table association
resource "aws_route_table_association" "igw_route_table_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.igw_route_table.id
}

# Create NAT Gateway
resource "aws_internet_gateway" "gateway" {
  subnet_id         = aws_subnet.public_subnet.id
  allocation_id     = aws_eip.eip.id
  connectivity_type = "public"
}

# Associate IGW Route Table with private subnet and public gateway
resource "aws_route_table" "nat_route_table" {
  vpc_id   = aws_vpc.vpc.id
}

# Create NAT route
resource "aws_route" "nat_route" {
  nat_gateway_id         = aws_nat_gateway.nat.id
  route_table_id         = aws_route_table.nat_route_table.id
  destination_cidr_block = "0.0.0.0/0"
}

# Create NAT route table association
resource "aws_route_table_association" "nat_route_table_association" {
  subnet_id      = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.nat_route_table.id
}
```
### Encryption

It's highly recommended to encryption sensitive data with KMS keys.

```terraform
resource "aws_kms_key" "key" {
  description = "KMS"
  key_usage   = "ENCRYPT_DECRYPT"

  tags = var.tags
}
```

### IAM

To ensure proper security, Role-based Access Control (RBAC) is necessary for the registry service. In this case, we require three roles to ensure flawless operation of the service:

- registry_execution: this role is responsible for provisioning and executing the registry service.
- registry_task: this role is needed to run the registry service and has access permissions to the s3 bucket.
- registry_autoscale_role: this role is responsible for monitoring CloudWatch metrics and autoscaling the registry service.

It is important to carefully select the minimum set of permissions to be granted to each role, as I will not delve deeply into permission control in this document. 

Please note that the following template provides powerful permissions and should never be used in a production environment.

```json
# Template of creating IAM roles

# Create role
resource "aws_iam_role" "registry_execution" {
  name               = "registry-execution"
  assume_role_policy = data.aws_iam_policy_document.registry_execution_assume.json

  inline_policy {
    name   = "registry-execution"
    policy = data.aws_iam_policy_document.registry_execution.json
  }

  tags = var.tags
}

# Grant assume role priviledge
data "aws_iam_policy_document" "registry_execution_assume" {
  statement {
    effect  = "Allow"
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ecs-tasks.amazonaws.com"]
    }
  }
}

# Grant access to any other resources
data "aws_iam_policy_document" "registry_execution" {
  effect = "Allow"
    actions = [
      "*"
    ]
    resources = [
      "*"
    ]
}
```


### ECS

`ECS (Elastic Container Service)` allows for running containers without the need to define the underlying infrastructure. Containers can be launched by using [TaskDefinition](https://docs.aws.amazon.com/us_en/AmazonECS/latest/developerguide/task_definitions.html).

Here we claim `FARGATE` as the type of nodes for running containers.

```json
resource "aws_ecs_cluster" "cluster" {
  name = "cluster"
}

resource "aws_ecs_cluster_capacity_providers" "cluster_capacity_providers" {
  cluster_name = aws_ecs_cluster.cluster.name

  capacity_providers = ["FARGATE"]

  default_capacity_provider_strategy {
    base              = 10
    weight            = 1
    capacity_provider = "FARGATE"
  }
}
```


### S3

`S3 Bucket` is for persistent storage. You can use the following configuration to claim a bucket:

```json
resource "aws_s3_bucket" "bucket" {
  bucket        = "name"
}

resource "aws_s3_bucket_policy" "bucket_policy" {
  bucket = aws_s3_bucket.bucket.id
  policy = data.aws_iam_policy_document.bucket_policy.json
}

data "aws_iam_policy_document" "bucket_policy" {
  # This block of configuration is region specific
  statement {
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::{your_account_id}:root"]
    }

    actions = [
      "s3:PutObject"
    ]

    resources = [
      "${aws_s3_bucket.bucket.arn}/{path_prefix}/AWSLogs/{aws_account_id}/*"
    ]
  }

  statement {
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = [aws_iam_role.registry_task.arn]
    }

    actions = [
      "you decide"
    ]

    resources = [
      aws_s3_bucket.bucket.arn
    ]
  }

  statement {
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = [aws_iam_role.registry_task.arn]
    }

    actions = [
      "choose which actions are allowed"
    ]

    resources = [
      "${aws_s3_bucket.bucket.arn}/*"
    ]
  }
}
```

Note that the `S3` support `ELB` logging is `region-specific`, see details [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/enable-access-logging.html#attach-bucket-policy)


### Secrets

Sensitive data can be injected into your container when the container is initially started. If the secret is subsequently updated or rotated, the container will not receive the updated value automatically. You must either launch a new task or if your task is part of a service you can update the service and use the Force new deployment option to force the service to launch a fresh task.

> The Docker Registry may requires user credentials when accessing private repos, hence, we need to specify the access credentials. This can be done by using the `aws_secretsmanager_secret` resource, which will be encrypted/decrypted by `aws_kms_key` for better security.
> In addition, we can also pass secret environment values to the container with `aws_ssm_parameter` resource.

```json
resource "aws_kms_key" "application" {
  key_usage   = "ENCRYPT_DECRYPT"
}

resource "aws_secretsmanager_secret" "application" {
  name                    = "credentials"
  kms_key_id              = resource.aws_kms_key.application.key_id
  recovery_window_in_days = 0 # force deletion at infra tearing down
}

resource "aws_secretsmanager_secret_version" "application" {
  secret_id     = aws_secretsmanager_secret.application.id
  secret_string = "{your credentials}"
}

resource "aws_ssm_parameter" "creds" {
  name        = "creds"
  type        = "StringList"
  value       = "{creds values}"
  data_type   = "text"
  key_id      = resource.aws_kms_key.application.id
}

```


### Task Definition

To run an `ECS` task or service, we need to firstly define the task with `TaskDefinition`. Here's an example template for running the `registry`:

```json
data "aws_region" "current" {}

locals {
  registry_definition {
    cpu          = 512
    essential    = false
    name         = "registry"

    linuxParameters = {
      initProcessEnabled = true
    }

    image     = var.registry_image
    essential = true

    logConfiguration = {
      logDriver = "awslogs"
      options = {
        awslogs-region        = data.aws_region.current.name
        awslogs-group         = aws_cloudwatch_log_group.log_group.name
        awslogs-stream-prefix = "registry"
      }
    }

    environment = [
      // environment variables
      // see https://docs.docker.com/registry/configuration/
    ]

    portMappings = [
      {
        name          = "registry"
        protocol      = "tcp"
        containerPort = var.registry_port
        hostPort      = var.registry_port
      }
    ]
  })

resource "aws_ecs_task_definition" "registry" {
  family                   = "${var.organization}-${var.environment}-${var.service}"
  network_mode             = var.network_mode
  requires_compatibilities = ["FARGATE"]
  cpu                      = var.registry_cpu
  memory                   = var.registry_memory

  execution_role_arn = aws_iam_role.registry_execution.arn
  task_role_arn      = aws_iam_role.registry_task.arn

  container_definitions = jsonencode(
    local.registry_definition
  )

  volume {
    name = "vol"
  }

  depends_on = [
    aws_iam_user.user,
    aws_s3_bucket.bucket
  ]
}
```

Note: if you want to pull images from a private registry, the following block is required in the task definition:

```json
repositoryCredentials: {
  credentialsParameter: arn:aws:secretsmanager:region:aws_account_id:secret:secret_name
}
```

See [AWS docs](https://docs.aws.amazon.com/us_en/AmazonECS/latest/developerguide/private-auth.html)


### Security Groups

```json
resource "aws_security_group" "registry" {
  name   = "${var.organization}-${var.environment}-${var.service}"
  vpc_id = aws_vpc.vpc.id

  ingress {
    from_port   = var.registry_port
    to_port     = var.registry_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 5000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  lifecycle {
    create_before_destroy = true
  }

  depends_on = [
    aws_vpc.vpc,
    aws_subnet.public_subnet,
    aws_subnet.private_subnet
  ]
}
```


### ALB

To enable public access to our `registry`, we need to define `ALB` for forwarding/distributing traffic.

```json
resource "aws_lb" "registry" {
  name = "${var.organization}-${var.environment}-${var.service}-registry"

  internal           = false
  load_balancer_type = "application"

  security_groups = [
    aws_security_group.registry.id,
  ]
  subnets = [aws_subnet.public_subnet.id]

  access_logs {
    bucket  = aws_s3_bucket.bucket.id
    prefix  = var.lb_log_prefix
    enabled = true
  }

  depends_on = [
    aws_s3_bucket.bucket,
    aws_s3_bucket_policy.bucket_policy
  ]
}

resource "aws_lb_target_group" "registry" {
  name                 = "${var.organization}-${var.environment}-${var.service}-registry"
  vpc_id               = aws_vpc.vpc.id
  port                 = var.registry_port
  protocol             = "HTTP"
  target_type          = var.lb_target_type
  deregistration_delay = 30

  health_check {
    enabled             = true
    interval            = 120
    path                = "/"
    protocol            = "HTTP"
    timeout             = 60
    matcher             = "200"
    unhealthy_threshold = 3
  }
}

resource "aws_lb_listener" "registry" {
  load_balancer_arn = aws_lb.registry.arn

  port            = var.registry_port
  protocol        = "HTTPS"
  ssl_policy      = "ELBSecurityPolicy-2016-08"
  certificate_arn = aws_acm_certificate.cert.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.registry.arn
  }

  depends_on = [
    aws_lb_target_group.registry,
    aws_acm_certificate.cert
  ]
}

resource "aws_lb_listener_rule" "registry" {
  listener_arn = aws_lb_listener.registry.arn

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.registry.arn
  }

  condition {
    source_ip {
      values = var.source_ip
    }
  }

  depends_on = [
    aws_lb_target_group.registry
  ]
}
```


### Service

Once we have the `TaskDefinition` and `ALB`, we can deploy the `registry` service:

```json
resource "aws_ecs_service" "registry" {
  name            = "registry"
  cluster         = aws_ecs_cluster.cluster.id
  task_definition = aws_ecs_task_definition.registry.arn
  desired_count   = 1

  enable_execute_command = false
  force_new_deployment   = true

  deployment_circuit_breaker {
    enable   = true
    rollback = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_truegroup.registry.arn
    container_name   = "registry"
    container_port   = var.registry_port
  }

  network_configuration {
    subnets          = [aws_subnet.private_subnet.id]
    assign_public_ip = false # set to false if using awsvpc network mode
    security_groups = [
      aws_security_group.registry.id,
    ]
  }

  wait_for_steady_state = true

  platform_version    = "LATEST"
  scheduling_strategy = "REPLICA"

  capacity_provider_strategy {
    capacity_provider = var.fargate_spot_enabled ? "FARGATE_SPOT" : "FARGATE"
    weight            = 10
  }

  timeouts {
    create = "5m"
    update = "5m"
    delete = "5m"
  }

  depends_on = [
    aws_lb.registry
  ]
}
```


### CloudWatch Metrics

We can monitor our `registry` service with `CloudWatch`:

```json
resource "aws_cloudwatch_log_group" "log_group" {
  name              = "${var.organization}-${var.environment}-${var.service}"
  retention_in_days = var.registry_cloudwatch_log_retention
}

resource "aws_cloudwatch_metric_alarm" "registry_target_replica_count" {
  alarm_name          = "registry-target-replica-count"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 2
  metric_name         = "HealthyHostCount"
  namespace           = "AWS/ApplicationELB"
  period              = 30
  statistic           = "Minimum"
  threshold           = var.registry_replica_count
  actions_enabled     = "true"
  alarm_actions       = [aws_sns_topic.registry.arn]
  alarm_description   = "This metric checks for registry container replica counts."
  dimensions = {
    LoadBalancer = aws_lb.registry.arn_suffix
    TargetGroup  = aws_lb_target_group.registry.arn_suffix
  }
}

resource "aws_cloudwatch_metric_alarm" "registry_target_cpu_alarm" {
  alarm_name          = "registry-target-cpu-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = 10
  statistic           = "Maximum"
  threshold           = 80
  actions_enabled     = true
  alarm_description   = "Alarm when CPU exceeds 80%"
  alarm_actions = [
    aws_sns_topic.registry.arn,
    aws_appautoscaling_policy.registry_cpu.arn
  ]
  dimensions = {
    ClusterName = aws_ecs_cluster.cluster.name
    ServiceName = aws_ecs_service.registry.name
  }

  tags = var.tags
}

resource "aws_sns_topic" "registry" {
  name = "${var.organization}-${var.environment}-${var.service}-registry"
}
```


### AutoScaling

We can also enable the `AutoScaling` feature for our cluster when detect heavy loads of pulling/caching.

```json
resource "aws_appautoscaling_target" "registry" {
  max_capacity       = var.registry_replica_count + 1
  min_capacity       = var.registry_replica_count
  resource_id        = "service/${aws_ecs_cluster.cluster.name}/${aws_ecs_service.registry.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
  role_arn           = aws_iam_role.registry-autoscale-role.arn

  depends_on = [
    aws_ecs_service.registry,
    aws_iam_role.registry-autoscale-role
  ]
}

resource "aws_appautoscaling_policy" "registry_cpu" {
  name               = "${var.organization}-${var.environment}-${var.service}-cpu-policy"
  policy_type        = "StepScaling"
  resource_id        = aws_appautoscaling_target.registry.resource_id
  scalable_dimension = aws_appautoscaling_target.registry.scalable_dimension
  service_namespace  = aws_appautoscaling_target.registry.service_namespace

  step_scaling_policy_configuration {
    adjustment_type          = "ChangeInCapacity"
    cooldown                 = 300
    metric_aggregation_type  = "Maximum"

    step_adjustment {
      metric_interval_lower_bound = -10.0
      metric_interval_upper_bound = 10.0
      scaling_adjustment          = 0
    }

    step_adjustment {
      metric_interval_upper_bound = -10.0
      scaling_adjustment          = -1
    }

    step_adjustment {
      metric_interval_lower_bound = 10.0
      scaling_adjustment          = 1
    }
  }
}
```


### ACM

Finally, we need to obtain a cert for our `ALB` to enable `HTTPS` support.

```json
data "aws_route53_zone" "zone" {
  name = "${var.environment}.${var.domain}"
}

resource "aws_acm_certificate" "cert" {
  domain_name       = "${var.service}.${var.environment}.${var.domain}"
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_route53_record" "acm_verification" {
  for_each = { for dvo in aws_acm_certificate.cert.domain_validation_options :
    dvo.domain_name => {
      domain_name = dvo.domain_name
      name        = dvo.resource_record_name
      record      = dvo.resource_record_value
      type        = dvo.resource_record_type
    }
  }

  allow_overwrite = true

  zone_id = data.aws_route53_zone.zone.zone_id
  name    = each.value.name
  records = [each.value.record]
  ttl     = 60
  type    = each.value.type
}

resource "aws_route53_record" "registry" {
  name    = "${var.service}.${var.environment}.${var.domain}"
  zone_id = data.aws_route53_zone.zone.id
  type    = "A"
  alias {
    name                   = aws_lb.registry.dns_name
    zone_id                = aws_lb.registry.zone_id
    evaluate_target_health = true
  }
}
```


## Conclusion

Voila, if you have everything properly configured, you can have a `docker pull-through cache proxy` and be able to get rid of the `fair use policy`, at least temporarily.

If you use the official `distribution` image for running the `registry`
---
title: "How to telegraf agent as daemon in ECS cluster"
date: 2023-09-24T18:30:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "ecs", "prometheus", "observability"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "How to telegraf agent as daemon in ECS cluster"
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://avatars.githubusercontent.com/in/94907?v=4" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/jcajkovic.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


# Deploying Telegraf Agent in a Docker Container on AWS ECS as a Daemon Service with Prometheus Integration

In this blog post, we will explore how to deploy a Telegraf agent in a Docker container on AWS Elastic Container Service (ECS) as a Daemon service. Additionally, we will configure Telegraf to collect metrics and expose them using the Prometheus client output plugin. This setup will allow you to easily collect and scrape Telegraf metrics for monitoring and alerting purposes.

## Table of Contents

1. **Introduction**
2. **Prerequisites**
3. **Create a Telegraf Configuration Template**
4. **Using confd for Dynamic Configuration**
5. **Dockerizing Telegraf with Dynamic Configuration**
6. **Configuring Telegraf for Prometheus**
7. **Creating a Docker Image**
8. **Deploying Telegraf on AWS ECS**
9. **Terraform Setup**
10. **Conclusion**

## 1. Introduction

Telegraf is a powerful open-source agent for collecting metrics and data from various sources, and it can integrate seamlessly with Prometheus, a popular open-source monitoring and alerting solution. In this guide, we will combine the capabilities of Telegraf and ECS to deploy a scalable metrics collection solution that exposes metrics in a format compatible with Prometheus.

## 2. Prerequisites

Before we begin, ensure you have the following prerequisites in place:

- An AWS account with appropriate permissions to create ECS services and resources.
- Terraform installed on your local machine.
- Basic familiarity with Docker, ECS, Telegraf, and Prometheus.

## 3. Create a Telegraf Configuration Template

Let's start by creating a template for the Telegraf configuration file (`telegraf.conf`). This template will allow us to inject dynamic configuration values during runtime.

```toml
# telegraf_template.conf

[agent]
  interval = {{ key "/telegraf/interval" }}

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[outputs.prometheus_client]]
  listen = ":9126" # Prometheus scrape endpoint
  metric_family = "telegraf"
  metric_name = "telegraf_metrics"
```

In this template, we use placeholders like `{{ key "/telegraf/interval" }}` that will be replaced with actual values using confd.

## 4. Using confd for Dynamic Configuration

Confd is a tool for managing configuration files using templates and data from various backends, such as etcd, Consul, or environment variables. We'll use confd to generate a dynamic Telegraf configuration.

Here's an example confd configuration (`telegraf.toml`) for our Telegraf setup:

```toml
[template]
src = "telegraf_template.conf"
dest = "/etc/telegraf/telegraf.conf"
keys = [
  "/telegraf/interval",
]
```

This configuration specifies the source template, destination file, and the keys that confd will use to fetch values from the backend and populate the template.

## 5. Dockerizing Telegraf with Dynamic Configuration

To containerize Telegraf with the dynamic configuration, we'll create a Dockerfile like this:

```Dockerfile
# Dockerfile

FROM telegraf:latest

# Install confd
RUN apt-get update && apt-get install -y confd

# Copy Telegraf configuration template and confd configuration
COPY telegraf_template.conf /etc/telegraf/telegraf_template.conf
COPY telegraf.toml /etc/confd/conf.d/telegraf.toml
COPY telegraf.toml.toml /etc/confd/templates/telegraf.toml.toml

# Entrypoint: Run confd to generate dynamic Telegraf config
CMD confd -onetime -backend env && telegraf -config /etc/telegraf/telegraf.conf
```

This Dockerfile sets up Telegraf, installs confd, and configures it to generate the Telegraf configuration file at runtime.

## 6. Configuring Telegraf for Prometheus

We've configured Telegraf to expose metrics using the Prometheus client output plugin. The `listen` parameter in the configuration template specifies the endpoint where Prometheus can scrape Telegraf metrics. You can modify this configuration to suit your needs.

## 7. Creating a Docker Image

Build the Docker image using the following command:

```bash
docker build -t my-telegraf .
```

## 8. Deploying Telegraf on AWS ECS

Now, let's deploy Telegraf as a Daemon service on AWS ECS, making it accessible for Prometheus to scrape metrics.

## 9. Terraform Setup

Here's a simplified Terraform configuration for deploying Telegraf on ECS with Prometheus integration. Save it in a file named `ecs.tf`.

terraform locals (locals.tf)

```tf
locals {
  container_registry = "${aws_account_id}.dkr.ecr.eu-west-1.amazonaws.com/monitoring/telegraf-daemon"
  telegraf_image = "${local.container_registry}:${var.telegraf_image_version}"
}
```

terraform data (data.tf)

```tf
data "aws_vpc" "selected" {
  id = var.vpc_id
}
```
terraform variables (variables.tf)

```terraform
variable "aws_region" { default = "eu-west-1" }

variable "stage" {
  type        = string
  description = "Stage variable"
}

variable "service" {
  type        = string
  description = "Name of the service"
}

variable "cluster_name" {
  type        = string
  description = "Name of the ECS cluster"
}

variable "vpc_id" {
  type        = string
  description = "VPC ID"
}

variable "subnets" {
  type        = list(string)
  description = "List of subnets"
}

variable "telegraf_image_version" {
  type        = string
  description = "AWS ECR repository where the image is stored with the proper tag"
  default = "latest"
}

variable "region" {
  type        = string
  description = "Default region"
  default     = "eu-west-1"
}

variable "container_cpu" {
  type        = number
  description = "Value for the container cpu configuration"
  default     = 256
}

variable "container_mem" {
  type        = number
  description = "Value for the container memory configuration"
  default     = 256
}

variable "status_port" {
  type        = number
  description = "Port number for the ecs service task health check"
  default     = 9273
}

variable "desired_count" {
  type        = number
  description = "Desired count for the ecs service"
  default     = 1
}
variable "service_sg_rules" {
  default = {
    egress = {
      "r01" = { from_port : 443, to_port : 443, protocol : "TCP", cidr_blocks : ["0.0.0.0/0"], description : "Allow connection to aws services" }
    }
  }
}


```

ECS resources (_ecs.tf)

```tf
resource "aws_cloudwatch_log_group" "telegraf-daemon-logroup" {
  name              = "${var.stage}-${var.service}-lg"
  retention_in_days = 7
  tags              = local.tags
}

resource "aws_ecs_task_definition" "telegraf-daemon-ecs-task" {
  depends_on         = [aws_iam_role.telegraf-daemon-ecs-role]
  family             = "${var.stage}-${var.service}-task"
  task_role_arn      = aws_iam_role.telegraf-daemon-ecs-role.arn
  execution_role_arn = aws_iam_role.telegraf-daemon-execution-role.arn
  network_mode = "awsvpc"
  volume {
    name = "root"
    host_path = "/"
  }
  volume {
    name = "var_run"
    host_path = "/var/run"
  }
  volume {
    name = "sys"
    host_path = "/sys"
  }
  volume {
    name = "var_lib_docker"
    host_path = "/var/lib/docker/"
  }
  container_definitions = jsonencode([
    {
      name      = "${var.stage}-${var.service}"
      image     = local.telegraf_image
      cpu       = var.container_cpu
      memory    = var.container_mem
      essential = true
      dockerLabels = {
        "PROMETHEUS_EXPORTER_PORT" : "9273"
      }
      portMappings = [
        {
          containerPort = 9273
          protocol      = "tcp"
        }
      ]
      environment = [
        { "name" : "HOST_ETC", "value" : "/rootfs/etc" },
        { "name" : "HOST_PROC", "value" : "/rootfs/proc" },
        { "name" : "HOST_SYS", "value" : "/rootfs/sys" },
        { "name" : "HOST_VAR", "value" : "/rootfs/var" },
        { "name" : "HOST_RUN", "value" : "/rootfs/run" },
        { "name" : "HOST_MOUNT_PREFIX", "value" : "/rootfs" }
      ]
      mountPoints = [
        {
          "sourceVolume" : "root",
          "containerPath" : "/rootfs",
          "readOnly" : true
        },
        {
          "sourceVolume" : "var_run",
          "containerPath" : "/var/run",
          "readOnly" : false
        },
        {
          "sourceVolume" : "sys",
          "containerPath" : "/sys",
          "readOnly" : true
        },
        {
          "sourceVolume" : "var_lib_docker",
          "containerPath" : "/var/lib/docker",
          "readOnly" : true
        }
      ]

      logConfiguration = {
        logDriver : "awslogs",
        options : {
          awslogs-group : aws_cloudwatch_log_group.telegraf-daemon-logroup.name
          awslogs-region : var.region
          awslogs-stream-prefix : var.service
          mode : "non-blocking"
        }
      }
    }
  ])
}

resource "aws_ecs_service" "telegraf-daemon-service" {
  name            = "${var.stage}-${var.service}-service"
  cluster         = var.cluster_name
  task_definition = aws_ecs_task_definition.telegraf-daemon-ecs-task.arn
  # desired_count          = var.desired_count
  enable_execute_command = true
  scheduling_strategy    = "DAEMON"
  launch_type            = "EC2"
  network_configuration {
    assign_public_ip = false
    security_groups  = [aws_security_group.telegraf-daemon-sg.id]
    subnets          = var.subnets
  }
  tags = local.tags
}

```

IAM resources (_iam.tf)

```terraform

resource "aws_iam_role" "telegraf-daemon-execution-role" {
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      },
    ]
  })

  tags = local.tags
}

resource "aws_iam_role" "telegraf-daemon-ecs-role" {
  name = "${var.stage}-${var.service}-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
  inline_policy {
    name = "ecr"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "ecr:ListImages",
            "ecr:GetDownloadUrlForLayer",
            "ecr:BatchGetImage",
            "ecr:BatchCheckLayerAvailability",
            "ecr:ListImages"
          ]
          Effect   = "Allow"
          Resource = "arn:aws:ecr:eu-west-1:522044537874:dkr.ecr/monitoring/telegraf-agent:*"
        },
      ]
    })
  }
  
  
  inline_policy {
    name = "cloudwatch"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "cloudwatch:PutMetricData",
            "cloudwatch:ListMetrics",
            "cloudwatch:GetMetricData",
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents",
            "logs:DescribeLogStreams"
          ]
          Effect   = "Allow"
          Resource = "*"
        }
      ]
    })
  }
  
  inline_policy {
    name = "autoscaling-ecs-service"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "application-autoscaling:*",
            "ecs:DescribeServices",
            "ecs:UpdateService",
            "iam:CreateServiceLinkedRole"
          ]
          Effect   = "Allow"
          Resource = "*"
        }
      ]
    })
  }

  inline_policy {
    name = "ecs_list_describe"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "ecs:ListServices",
            "ecs:DescribeServices",
            "ecs:DescribeClusters",
            "ecs:ListClusters",
          ]
          Effect   = "Allow"
          Resource = "*"
        }
      ]
    })
  }
  inline_policy {
    name = "entrypoint-policy"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "ec2:DescribeInstances",
            "iam:ListAccountAliases",
          ]
          Effect   = "Allow"
          Resource = "*"
        }
      ]
    })
  }
  tags = local.tags
}

resource "aws_iam_role_policy_attachment" "ecs-task-execution-role-policy-attachment" {
  role       = aws_iam_role.telegraf-daemon-execution-role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}


```

network resources (_network.tf)

```hcl
resource "aws_security_group" "telegraf-daemon-sg" {
  name        = "${var.stage}-${var.service}-sg"
  description = "Security group for telegraf VPC probe"
  vpc_id      = data.aws_vpc.selected.id
  tags = merge(
    local.tags,
    {
      Name = "${var.stage}-${var.service}-sg"
    },
  )
}

resource "aws_security_group_rule" "service_sg_ingress_rules" {
  for_each = var.service_sg_rules.egress

  type              = "egress"
  from_port         = each.value.from_port
  to_port           = each.value.to_port
  protocol          = each.value.protocol
  cidr_blocks       = each.value.cidr_blocks
  description       = each.value.description
  security_group_id = aws_security_group.telegraf-daemon-sg.id
}


resource "aws_security_group_rule" "all-ports-in" {
  type              = "ingress"
  from_port         = 0
  to_port           = 65535
  protocol          = "TCP"
  cidr_blocks       = [data.aws_vpc.selected.cidr_block]
  security_group_id = aws_security_group.telegraf-daemon-sg.id
  description       = "Allow connection on all TCP ports from the VPC"
}

```

Run the following commands to initialize and apply the Terraform configuration:

```bash
terraform init
terraform apply
```

This Terraform code sets up an ECS cluster, defines a task for Telegraf, and deploys it as a service.

## 10. Conclusion

In this blog post, we explored how to deploy a Telegraf agent in a Docker container on AWS ECS as a Daemon service while configuring it to expose metrics using the Prometheus client output plugin. This setup allows you to easily collect and scrape Telegraf metrics for monitoring and alerting purposes, making it a valuable addition to your containerized applications in AWS ECS.
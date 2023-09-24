---
title: "How to service discovery with Prometheus on AWS ECS"
date: 2023-09-24T11:30:03+00:00
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
description: "How to service discovery with Prometheus on AWS ECS"
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
    image: "https://www.vectorlogo.zone/logos/prometheusio/prometheusio-ar21.svg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/jcajkovic.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# How to service discovery with promehteus on AWS ECS

Service discovery is a critical component of modern containerized applications, ensuring that your monitoring and observability tools can automatically discover and track the various services and instances in your infrastructure. Amazon Elastic Container Service (ECS) is a popular container orchestration service that simplifies deploying and managing containers. When running Prometheus in an ECS cluster, it's essential to have a robust service discovery mechanism in place. In this blog post, we'll explore how to set up service discovery for Amazon ECS with Prometheus running in a Fargate container using the tkgregory/prometheus-ecs-discovery sidecar.

## Understanding Service Discovery

Service discovery allows monitoring tools like Prometheus to dynamically discover and scrape metrics from various containers, services, and instances, without requiring manual configuration. In the context of Amazon ECS, tasks are ephemeral and can be started and stopped frequently. As a result, static configuration for Prometheus becomes cumbersome and impractical.

## Prometheus and ECS: A Challenging Combination

Prometheus is a popular open-source monitoring and alerting toolkit designed for reliability and scalability. While it excels at collecting and querying metrics, setting up Prometheus to monitor services within an Amazon ECS cluster can be challenging due to the dynamic nature of ECS tasks. This is where service discovery solutions come into play.

## Enter tkgregory/prometheus-ecs-discovery

[tkgregory/prometheus-ecs-discovery](https://github.com/tkgregory/prometheus-ecs-discovery) is a Prometheus service discovery plugin specifically built for Amazon ECS. This tool makes it easier to discover and scrape metrics from ECS tasks without the need for manual configuration updates each time a task starts or stops.

Here's how tkgregory/prometheus-ecs-discovery works:

1. **ECS Service Discovery**: ECS itself provides service discovery through DNS for the tasks in your cluster. Each task gets a DNS entry in the form of `<task-name>.<service-name>.<task-id>.<region>.ecs.amazonaws.com`. tkgregory/prometheus-ecs-discovery leverages this feature to discover tasks.

2. **Sidecar Container**: To use tkgregory/prometheus-ecs-discovery, you deploy it as a sidecar container alongside your Prometheus container in the same task definition. This sidecar queries the ECS metadata service to discover active tasks and exports them to Prometheus in a format Prometheus understands.

## Setting Up Prometheus with tkgregory/prometheus-ecs-discovery

Now, let's walk through the steps to set up Prometheus with tkgregory/prometheus-ecs-discovery for service discovery in Amazon ECS:

### 1. Create an ECS Task Definition

First, you need to create an ECS task definition that includes both your Prometheus container and the tkgregory/prometheus-ecs-discovery sidecar container.

Here's a simplified example of a task definition in JSON format:

```json
{
  "family": "prometheus",
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "prometheus",
      "image": "prom/prometheus",
      "cpu": 0,
      "portMappings": [
        {
          "containerPort": 9090,
          "hostPort": 9090
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "config",
          "containerPath": "/output",
          "readOnly": false
        }
      ]
    },
    {
      "name": "prometheus-ecs-discovery",
      "image": "tkgregory/prometheus-ecs-discovery",
      "cpu": 0,
      "portMappings": [],
      "essential": false,
      "mountPoints": [
        {
          "sourceVolume": "config",
          "containerPath": "/output",
          "readOnly": false
        }
      ]
    }
  ],
  "volumes": [
    {
      "name": "config",
      "host": {}
    }
  ],
  "networkMode": "awsvpc"
}
```

In this example, the `prometheus` container runs the Prometheus server, and the `prometheus-ecs-discovery` container is the sidecar.

Here is an example terraform code:

```hcl

### DATA Resources

data "aws_vpc" "selected" {
  id = var.vpc_id
}

data "aws_region" "current" {}

### IAM Resources

data "aws_iam_policy_document" "prometheus_iam_policy_document" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["ecs-tasks.amazonaws.com"]
    }
  }
}

resource "aws_iam_role" "PrometheusExecutionRole" {
  name                = "prometheus-execution-role"
  assume_role_policy  = data.aws_iam_policy_document.prometheus_iam_policy_document.json
  managed_policy_arns = ["arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"]
}

#tfsec:ignore:aws-iam-no-policy-wildcards
resource "aws_iam_role" "PrometheusRole" {
  name                = "prometheus-role"
  assume_role_policy  = data.aws_iam_policy_document.prometheus_iam_policy_document.json
  managed_policy_arns = ["arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceEventsRole"]

  inline_policy {
    name = "prometheus-policy"

    policy = jsonencode({
      Version = "2012-10-17",
      Statement = [
        {
          Action = [
            "ecs:ListClusters",
            "ecs:ListTasks",
            "ecs:DescribeTask",
            "ecs:DescribeContainerInstances",
            "ecs:DescribeTasks",
            "ecs:DescribeTaskDefinition",
            "ec2:DescribeInstances",
            "ec2:DescribeAvailabilityZones"
          ],
          Effect   = "Allow",
          Resource = "*",
        }
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
            "logs:DescribeLogGroups",
            "logs:CreateLogStream",
            "logs:DescribeLogStreams",
            "logs:PutLogEvents"
          ]
          Effect   = "Allow"
          Resource = "*"
        },
      ]
    })
  }

}

### ECS Resources

resource "aws_ecs_task_definition" "PrometheusTaskDefinition" {
  family                   = "prometheus-for-ecs"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]

  execution_role_arn = aws_iam_role.PrometheusExecutionRole.arn
  task_role_arn      = aws_iam_role.PrometheusRole.arn

  cpu    = 256
  memory = 512

  volume {
    name = "config"
  }

  container_definitions = jsonencode([
    {
      name = "prometheus-for-ecs"
      image = "tkgregory/prometheus-with-remote-configuration:latest"

      dockerLabels = {
        "PROMETHEUS_EXPORTER_PORT" : "9090"
      }

      port_mappings = [
        {
          containerPort = 9090
        }
      ]

      mountPoints = [
        {
          "sourceVolume" : "config",
          "containerPath" : "/output",
          "readOnly" : false
        }
      ]

      logConfiguration = {
        logDriver : "awslogs",

        options : {
          awslogs-group : aws_cloudwatch_log_group.PrometheusLogGroup.name
          awslogs-region : data.aws_region.current.name
          awslogs-stream-prefix : "prometheus-ecs-discovery"
          mode : "non-blocking"
        }
      }
    },
    {
      name  = "prometheus-ecs-discovery"
      image = "tkgregory/prometheus-ecs-discovery:latest"

      command = ["-config.write-to=/output/ecs_file_sd.yml"]

      environment = [
        {
          name  = "AWS_REGION"
          value = data.aws_region.current.name
        },
      ]

      mountPoints = [
        {
          "sourceVolume" : "config",
          "containerPath" : "/output",
          "readOnly" : false
        }
      ]

      logConfiguration = {
        logDriver : "awslogs",

        options : {
          awslogs-group : aws_cloudwatch_log_group.PrometheusLogGroup.name
          awslogs-region : data.aws_region.current.name
          awslogs-stream-prefix : "prometheus-ecs-discovery"
          mode : "non-blocking"
        }
      }
    },
  ])
}

resource "aws_ecs_service" "PrometheusService" {
  name            = "prometheus-service"
  cluster         = var.cluster_name
  task_definition = aws_ecs_task_definition.PrometheusTaskDefinition.arn
  launch_type     = "FARGATE"
  desired_count   = 1

  network_configuration {
    # subnets = [aws_subnet.PublicSubnet.id]
    subnets          = var.subnets
    security_groups  = [aws_security_group.PrometheusSecurityGroup.id]
    assign_public_ip = true
  }
}

#tfsec:ignore:aws-ec2-no-public-ingress-sgr
#tfsec:ignore:aws-ec2-no-public-egress-sgr
resource "aws_security_group" "PrometheusSecurityGroup" {
  name        = "PrometheusSecurityGroup"
  description = "Security group for Prometheus"
  vpc_id      = var.vpc_id

}

#tfsec:ignore:aws-ec2-no-public-ingress-sgr
resource "aws_security_group_rule" "prometheus_port_ingress_rule" {
  type              = "ingress"
  from_port         = 9090
  to_port           = 9090
  protocol          = "TCP"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.PrometheusSecurityGroup.id
  description       = "Allow inbound traffic on 9090 port from public internet"
  # cidr_blocks       = [data.aws_vpc.selected.cidr_block]
}

resource "aws_security_group_rule" "all_ports_to_vpc_sg_egress_rule" {
  type              = "egress"
  from_port         = 0
  to_port           = 65535
  protocol          = "tcp"
  cidr_blocks       = [data.aws_vpc.selected.cidr_block]
  description       = "Allow outbound traffic on all ports in VPC"
  security_group_id = aws_security_group.PrometheusSecurityGroup.id
}

#tfsec:ignore:aws-ec2-no-public-egress-sgr
resource "aws_security_group_rule" "https_to_public_sg_egress_rule" {
  type              = "egress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "Allow HTTPS outbound traffic to public internet"
  security_group_id = aws_security_group.PrometheusSecurityGroup.id
}

#tfsec:ignore:aws-cloudwatch-log-group-customer-key
resource "aws_cloudwatch_log_group" "PrometheusLogGroup" {
  name              = "prometheus"
  retention_in_days = 7
}

### Variables

variable "cluster_name" {
  type = string
  description = "(Mandatory) Value of cluster name where prometheus will be deployed"
}

variable "grafana_cloud_url" {
  type = string
  description = "(Mandatory) Value of grafana cloud url where prometheus will send metrics"
}

variable "prometheus_token" {
  type = string
  description = "(Mandatory) Value of prometheus token (for grafana cloud)"
}

variable "prometheus_user" {
  type = string
  description = "(Mandatory) Value of prometheus user (for grafana cloud)"
}
  
variable "subnets" {
  type = list(string)
}
  
variable "vpc_id" {
  type = string
}

```

### 2. Configure Prometheus

Configure Prometheus to use the tkgregory/prometheus-ecs-discovery service discovery mechanism. You can do this in your Prometheus configuration file by adding a job with the ECS service discovery configuration:

```yaml
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'ecs'
    file_sd_configs:
      - files:
          - /output/ecs_file_sd.yml

```

### 3. Deploy to ECS

Deploy your task definition to an ECS cluster as a service, ensuring that both the Prometheus and tkgregory/prometheus-ecs-discovery containers run together.

### 4. Verify Service Discovery

Once your ECS service is up and running, Prometheus should automatically discover and scrape metrics from the tasks in your cluster without any manual intervention.

## Conclusion

Service discovery is essential for monitoring containerized applications running in Amazon ECS. Using tkgregory/prometheus-ecs-discovery as a sidecar container alongside Prometheus simplifies the process of discovering and scraping metrics from ECS tasks. With this setup, your monitoring solution can adapt to the dynamic nature of ECS, ensuring that you capture all the relevant metrics for your containerized services.
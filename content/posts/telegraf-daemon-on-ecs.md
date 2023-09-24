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

```hcl
provider "aws" {
  region = "us-east-1"  # Change this to your desired region
}

resource "aws_ecs_cluster" "telegraf_cluster" {
  name = "telegraf-cluster"
}

resource "aws_ecs_task_definition" "telegraf_task" {
  family                   = "telegraf-task"
  network_mode             = "awsvpc"

  # Specify the Docker image you built earlier
  container_definitions = <<EOF
[
  {
    "name": "telegraf-container",
    "image": "my-telegraf:latest",
    "essential": true
  }
]
EOF
}

resource "aws_ecs_service" "telegraf_service" {
  name            = "telegraf-service"
  cluster         = aws_ecs_cluster.telegraf_cluster.id
  task_definition = aws_ecs_task_definition.telegraf_task.arn
  launch_type     = "EC2"
  desired_count   = 1  # Adjust as needed
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
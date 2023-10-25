---
title: "Creating Multi-Architectural Docker Images for AWS ECR"
date: 2023-03-17T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "docker", "multiarchitecture"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how to build and deploy multi-architectural Docker images (amd64 and arm64) to AWS ECR using the `docker manifest` command."
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
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/jcajkovic.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Creating Multi-Architectural Docker Images for AWS ECR

Amazon's Elastic Container Registry (ECR) is a fully-managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images. In this post, we'll walk through how to build multi-architectural Docker images for both `amd64` and `arm64` architectures, and then push them to AWS ECR using the `docker manifest` command.

## Prerequisites:

- Docker 19.03 or later installed.
- AWS CLI configured with appropriate permissions.
- An AWS ECR repository.

## Step 1: Login to AWS ECR

First, retrieve an authentication token and authenticate your Docker client to your registry:

```bash
aws ecr get-login-password --region region | docker login --username AWS --password-stdin your-account-id.dkr.ecr.region.amazonaws.com
```

Replace `region` with your AWS region and `your-account-id` with your AWS account ID.

## Step 2: Build the Docker images for each architecture

For `amd64`:

```bash
docker build -t your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-amd64 .
```

For `arm64` (on an ARM machine or using cross-compilation):

```bash
docker build -t your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-arm64 .
```

## Step 3: Push the Docker images to AWS ECR

For `amd64`:

```bash
docker push your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-amd64
```

For `arm64`:

```bash
docker push your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-arm64
```

## Step 4: Create a Docker manifest for the multi-architectural image

Before creating the manifest, ensure you've enabled experimental CLI features:

```bash
export DOCKER_CLI_EXPERIMENTAL=enabled
```

Now, create the manifest:

```bash
docker manifest create your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:latest \
your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-amd64 \
your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-arm64
```

## Step 5: Annotate the manifest with the architecture information

```bash
docker manifest annotate your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:latest \
your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-amd64 --arch amd64

docker manifest annotate your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:latest \
your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:tag-arm64 --arch arm64
```

## Step 6: Push the manifest to AWS ECR

```bash
docker manifest push your-account-id.dkr.ecr.region.amazonaws.com/your-repo-name:latest
```

## Conclusion

You've now successfully created a multi-architectural Docker image for both `amd64` and `arm64` architectures and pushed it to AWS ECR. By using Docker's manifest feature, you can ensure that the correct image is pulled based on the architecture of the machine.

---

I hope that helps! This is a concise guide, so ensure to adapt and test the steps according to your actual environment and requirements.
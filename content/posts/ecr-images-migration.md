---
title: "How to Easily Cross-Account Migrate Images Between ECR Repositories"
date: 2024-12-10T18:00:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "ecr", "bash"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A comprehensive guide on how to migrate container images between AWS ECR repositories across different accounts using AWS CLI and Docker."
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


# How to Easily Cross-Account Migrate Images Between ECR Repositories

Migrating container images across AWS Elastic Container Registry (ECR) repositories in different accounts is a common task for teams managing multi-account setups. This guide outlines a simple approach to migrate images using the AWS CLI and Docker.

## Prerequisites

Before you begin, ensure the following:

1. **AWS CLI Installed**  
   The AWS Command Line Interface (CLI) must be installed and configured on your machine. [Get AWS CLI here](https://aws.amazon.com/cli/).

2. **Docker Installed**  
   Docker must be installed and running on your machine. [Install Docker here](https://www.docker.com/products/docker-desktop).

3. **IAM Permissions**  
   Both source and destination accounts must have IAM roles or users with the necessary permissions:
   - **Source Account**: Permission to `describe-images` and `ecr:get-login-password`.
   - **Destination Account**: Permission to `ecr:batch-check-layer-availability`, `ecr:put-image`, and `ecr:get-login-password`.

4. **Repositories Exist**  
   Ensure the repositories in both source and destination accounts already exist.

5. **AWS Profiles Configured**  
   Use named profiles for each account in your `~/.aws/credentials` file for secure authentication. For example:
   - `source_account_profile` for the source account.
   - `destination_account_profile` for the destination account.

## Step-by-Step Migration Script

Here’s a script to migrate images from one ECR repository to another across accounts. Below, account details and repository names have been anonymized for security.

### The Migration Script


```bash
#!/bin/bash

# Variables
SOURCE_ACCOUNT_ID="source_account_id"
SOURCE_REGION="source_region"
DEST_ACCOUNT_ID="destination_account_id"
DEST_REGION="destination_region"

# List of repositories to process
REPOSITORIES=("repository_name")

# AWS CLI Login to both source and destination ECR
echo "Logging in to AWS ECR..."
aws ecr get-login-password --region $SOURCE_REGION --profile source_account_profile | docker login --username AWS --password-stdin $SOURCE_ACCOUNT_ID.dkr.ecr.$SOURCE_REGION.amazonaws.com
aws ecr get-login-password --region $DEST_REGION --profile destination_account_profile | docker login --username AWS --password-stdin $DEST_ACCOUNT_ID.dkr.ecr.$DEST_REGION.amazonaws.com

# Process each repository
for REPO in "${REPOSITORIES[@]}"; do
  echo "Processing repository: $REPO"

  # Fetch image tags from source repository
  echo "Fetching image tags from the source repository..."
  IMAGE_TAGS=$(aws ecr describe-images --repository-name $REPO --region $SOURCE_REGION --query 'imageDetails[].imageTags[]' --output text --profile source_account_profile)

  if [ -z "$IMAGE_TAGS" ]; then
    echo "No images found in the source repository for $REPO."
    continue
  fi

  # Migrate each image
  for TAG in $IMAGE_TAGS; do
    SOURCE_IMAGE="$SOURCE_ACCOUNT_ID.dkr.ecr.$SOURCE_REGION.amazonaws.com/$REPO:$TAG"
    DEST_IMAGE="$DEST_ACCOUNT_ID.dkr.ecr.$DEST_REGION.amazonaws.com/$REPO:$TAG"
    
    echo "Migrating image: $SOURCE_IMAGE to $DEST_IMAGE"
    
    # Pull the image from source
    docker pull $SOURCE_IMAGE
    
    # Tag the image for the destination
    docker tag $SOURCE_IMAGE $DEST_IMAGE
    
    # Push the image to the destination
    docker push $DEST_IMAGE
    
    # Optional: Remove local image
    # docker rmi $SOURCE_IMAGE $DEST_IMAGE
  done
done

echo "Migration completed successfully."

```

## How It Works

1. **Authentication**:
   The script logs into both source and destination ECR using `aws ecr get-login-password`. This authenticates Docker to pull and push images.

2. **Fetch Image Tags**:
   It retrieves all image tags in the source repository using `describe-images`.

3. **Pull, Tag, Push**:
   For each image tag:
   - The script pulls the image from the source repository.
   - Tags it for the destination repository.
   - Pushes it to the destination repository.

4. **Clean Up (Optional)**:
   You can remove local images after migration to save disk space.

## Key Notes

- **Error Handling**: Add error handling for scenarios like network interruptions or missing images to improve script resilience.
- **Large Migrations**: For repositories with a high number of tags, consider running the script in smaller batches.
- **Cross-Region Considerations**: The script supports migrations between different AWS regions by setting the `SOURCE_REGION` and `DEST_REGION` variables.

By following this approach, you can seamlessly migrate container images across ECR repositories, enabling smoother workflows in multi-account AWS environments.
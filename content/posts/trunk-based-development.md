---
title: "How Trunk Development Powers Continuous Delivery for Platform Engineers"
date: 2024-12-06T06:30:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["ai"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Explore how trunk-based development empowers platform engineers to achieve continuous delivery, focusing on the integration of small, incremental changes and the role of CI/CD tooling."
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

# How Trunk Development Powers Continuous Delivery for Platform Engineers

In the world of modern software development, speed and reliability are not optional—they’re the price of entry. For platform and operations engineers, particularly those working with tools like Terraform for infrastructure-as-code (IaC) and Python for AWS Lambda functions, trunk-based development is a game-changer. It aligns perfectly with the principles of continuous delivery (CD), allowing you to push small, tested, production-ready chunks of code within hours—sometimes even minutes—of writing them.

Let’s unpack how trunk development intersects with continuous delivery, the role of feature branches and rebasing, and the critical role of CI/CD tooling and tests.

## Trunk-Based Development: The Backbone of Continuous Delivery

At its core, trunk-based development focuses on collaboration and speed by keeping the mainline branch (often called main, master, or trunk) as the single source of truth. Engineers commit small, incremental changes directly to the trunk or short-lived feature branches that are quickly merged back.

For a platform/ops engineer:

- **Terraform Workflows**: Each change to your IaC might involve small adjustments to resource configurations, new modules, or updates to existing ones.
- **Lambda Development**: Your Python scripts might undergo minor tweaks or expansions, like adding a new handler for an AWS event.

These changes, small by design, are continuously integrated into the trunk, ensuring minimal divergence and fast feedback.

## Rebasing Feature Branches: Balancing Focused Work with Collaboration

While trunk-based development prioritizes direct commits to the main branch, there are scenarios—like experimental code or more complex changes—where a short-lived feature branch is necessary.

### Key Practices:

1. **Frequent Rebasing**: Keeping your feature branch up-to-date with the trunk minimizes integration pain. It ensures that your branch reflects the latest state of the system.
2. **Fast Merges**: Once the feature is complete and tested, it’s merged back into the trunk, ideally within hours or a day.

For example:

- When you’re iterating on a new Terraform module for VPC peering, you might work in a feature branch to validate it without disrupting the main configuration.
- Once you’re confident in your changes, you rebase to align with trunk and merge back quickly to deliver value.

This flow mirrors the essence of continuous delivery: working in small, deployable increments while keeping the mainline stable and always releasable.

## The Role of CI/CD Tooling: Automating the Pipeline

CI/CD tooling is the glue that holds trunk-based development and continuous delivery together. Tools like GitHub Actions, Jenkins, CircleCI, or AWS CodePipeline play an essential role in automating:

1. **Static Analysis**: Ensuring Terraform code adheres to best practices (e.g., terraform validate or tflint).
2. **Unit Tests**: Validating Python Lambda functions with a battery of tests to catch logic errors early.
3. **Integration Tests**: Running terraform plan in isolated environments to simulate infrastructure changes.
4. **Deployment**: Automatically pushing changes to production with tools like Github Actions, Atlantis for Terraform or AWS CDK Pipelines.

## Tests: The Bedrock of Fast, Reliable Deployments

Testing is non-negotiable in trunk-based development and continuous delivery. Without it, fast deployments would be reckless. For a platform engineer, testing typically includes:

1. **Unit Tests**:
   - For Python Lambdas, frameworks like pytest ensure logic works as expected.
   - For Terraform, using modules like terratest can validate configurations in test environments.
2. **Integration Tests**:
   - Simulating infrastructure changes in non-production environments.
   - Validating cross-resource dependencies, like ensuring an S3 bucket policy allows access only to the expected Lambda function.
3. **Smoke Tests**:
   - Verifying basic functionality immediately after deployment, such as ensuring a Lambda function runs successfully or an EC2 instance spins up correctly.
4. **Monitoring as Validation**:
   - Real-time checks using CloudWatch, Splunk, or Datadog to ensure deployments don’t introduce performance regressions.

Tests provide the safety net that allows you to push changes quickly and confidently.

## Why It Works for Platform Engineers

For a platform/ops engineer, trunk-based development aligns with the realities of managing infrastructure and backend systems:

- **Fast Feedback Loops**: You know within minutes if your Terraform plan introduced issues or your Lambda logic is broken.
- **Simplified Collaboration**: With a single source of truth, you avoid long-lived branches that lead to merge hell.
- **Incremental Progress**: Continuous delivery thrives on small, independent changes, which fits the modular nature of Terraform and Lambda development.

## Bringing It All Together

Trunk-based development isn’t just a buzzword—it’s a mindset that enables continuous delivery. For platform and ops engineers, the combination of small, frequent commits, robust CI/CD tooling, and automated testing transforms how you deliver value. You’re no longer shipping “big bang” releases; instead, you’re delivering incremental, reliable improvements that keep your systems evolving smoothly.

As an engineer working with Terraform and Python, the true power of this approach lies in its simplicity: ship small, ship fast, and ship often. With a trunk that’s always deployable and a pipeline that catches errors early, you can confidently bring your changes to production at the speed of thought.

## Takeaway

Trunk-based development isn’t just for app developers—it’s a perfect fit for platform and operations teams embracing continuous delivery. With CI/CD automation, rigorous testing, and a disciplined approach to small, incremental changes, you can achieve a streamlined, high-velocity delivery process that keeps your platforms reliable and your stakeholders happy.
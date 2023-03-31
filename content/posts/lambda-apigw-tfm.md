---
title: "Lambda behind API Gateway with terraform"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "lambda", "apigw"]
author: "Jozef Cajkovic"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Tutorial to create Lambda function accessible through API Gateway with terraform"
canonicalURL: "https://canonical.url/to/page"
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
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

### Step 1: Install Terraform

First, you need to install Terraform on your local machine. You can download the appropriate package for your operating system from the official website:

https://www.terraform.io/downloads.html

After downloading the package, extract it to a directory and add the directory to your system's PATH environment variable.

### Step 2: Create an IAM Role for the Lambda Function

The Lambda function requires an IAM role to access other AWS services, such as API Gateway. You can create an IAM role using Terraform by creating a new file with the .tf extension and adding the following code:

``` json
resource "aws_iam_role" "lambda_role" {
  name = "lambda_role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_role_policy" {
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
  role       = aws_iam_role.lambda_role.name
}
```

This code creates a new IAM role called lambda_role and attaches the AWSLambdaBasicExecutionRole policy to it, which provides basic permissions for Lambda functions to write logs to CloudWatch.

### Step 3: Create the Lambda Function

Next, you can create the Python Lambda function by creating a new file with the .tf extension and adding the following code:

``` kotlin

data "archive_file" "lambda_package" {
  type        = "zip"
  source_file = "lambda_function.py"
  output_path = "lambda_function.zip"
}

resource "aws_lambda_function" "lambda_function" {
  function_name    = "lambda_function"
  handler          = "lambda_function.lambda_handler"
  runtime          = "python3.8"
  memory_size      = 128
  timeout          = 30
  filename         = data.archive_file.lambda_package.output_path
  source_code_hash = data.archive_file.lambda_package.output_base64sha256
  
  role = aws_iam_role.lambda_role.arn

  environment {
    variables = {
      PARAM1 = "value1"
      PARAM2 = "value2"
    }
  }
}
```
This code creates a new Python Lambda function called lambda_function and specifies the handler function lambda_function.lambda_handler defined in a file called lambda_function.py. It also sets the IAM role for the function and sets some environment variables.

### Step 4: Create the API Gateway

You can create an API Gateway by adding the following code to a new .tf file:

```
resource "aws_api_gateway_rest_api" "example" {
  name = "example"
}

resource "aws_api_gateway_resource" "example" {
  rest_api_id = aws_api_gateway_rest_api.example.id
  parent_id   = aws_api_gateway_rest_api.example.root_resource_id
  path_part   = "example"
}

resource "aws_api_gateway_method" "example" {
  rest_api_id   = aws_api_gateway_rest_api.example.id
  resource_id   = aws_api_gateway_resource.example.id
  http_method   = "GET"
  authorization = "NONE"
}

resource "aws_api_gateway_integration" "example" {
  rest_api_id          = aws_api_gateway_rest_api.example.id
  resource_id          = aws_api_gateway_resource.example.id
  http_method          = "POST"
  type                 = "AWS_PROXY"
  uri                  = aws_lambda_function.lambda_function.invoke_arn
}

resource "aws_api_gateway_deployment" "example" {
  rest_api_id = aws_api_gateway_rest_api.example.id
  stage_name  = "prod"
}
```

This code creates an API Gateway REST API and sets up a new resource and method for the API. It also creates an integration between the API Gateway and the Lambda function using the AWS_PROXY type. Finally, it deploys the API Gateway to a new stage called prod.

### Step 5: Deploy the Lambda Function and API Gateway

To deploy the Lambda function and API Gateway, run the following commands in your Terraform project directory:

```bash
terraform init
terraform plan
terraform apply
```

These commands initialize Terraform, create an execution plan, and apply the changes to your AWS account.

### Step 6: Test the API Gateway

Once the deployment is complete, you can test the API Gateway by accessing the URL provided in the Terraform output. For example, if the output shows the URL as https://example.execute-api.us-east-1.amazonaws.com/prod, you can test it by sending a GET request to https://example.execute-api.us-east-1.amazonaws.com/prod/example.

That's it! You have successfully created and deployed a Python Lambda function connected to API Gateway with Terraform in AWS.
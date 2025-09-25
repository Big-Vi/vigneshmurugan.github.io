---
title: "ECS Task Role vs Task Execution Role: Understanding AWS Credential Injection and SDK Integration"
date: 2025-06-19T00:00:01Z
draft: false
summary: "A comprehensive guide explaining the difference between ECS Task Roles and Task Execution Roles, how AWS automatically injects credentials into containers, and how the AWS SDK discovers and uses these credentials for secure API access."
tags: ['aws', 'ecs', 'iam', 'security', 'containers', 'credentials', 'docker', 'fargate', 'ec2']
---

Amazon ECS (Elastic Container Service) provides two distinct IAM role mechanisms that often confuse developers: Task Roles and Task Execution Roles. Understanding the differences, use cases, and credential injection mechanisms is crucial for building secure containerized applications. This post explores both roles, examines how credentials are injected into containers, and demonstrates how the AWS SDK discovers and uses these credentials.

## Understanding the Two Role Types

### Task Execution Role vs Task Role Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    ECS Service Architecture                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                ECS Agent                           │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │         Task Execution Role                │   │   │
│  │  │   • Pull container images                  │   │   │
│  │  │   • Create CloudWatch log groups           │   │   │
│  │  │   • Register task definitions              │   │   │
│  │  │   • Network configuration                  │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┐   │
│                                                       │   │
│    ┌─────────────────────────────────────────────────┐ │   │
│    │              Container                          │ │   │
│    │  ┌─────────────────────────────────────────┐   │ │   │
│    │  │           Task Role                     │   │ │   │
│    │  │   • Application AWS API calls          │   │ │   │
│    │  │   • Access S3, DynamoDB, SQS, etc.     │   │ │   │
│    │  │   • Runtime permissions                │   │ │   │
│    │  └─────────────────────────────────────────┘   │ │   │
│    └─────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Task Execution Role: Infrastructure Permissions

The Task Execution Role is used by the ECS agent to set up and manage the container infrastructure.

**Key Responsibilities:**
- Pulling container images from ECR
- Creating and managing CloudWatch log groups
- Retrieving secrets from AWS Systems Manager Parameter Store or AWS Secrets Manager
- Setting up network interfaces
- Managing task lifecycle

**Required by:** ECS Agent (not your application)

### Task Role: Application Permissions

The Task Role provides AWS permissions to the application running inside the container.

**Key Responsibilities:**
- Making AWS API calls from application code
- Accessing AWS services like S3, DynamoDB, SQS, SNS
- Runtime permissions for business logic
- Cross-service authentication

**Required by:** Your application code (AWS SDK calls)

### Standard Task Execution Role Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

### Task Definition with Execution Role

```json
{
    "family": "my-app-task",
    "networkMode": "awsvpc",
    "requiresCompatibilities": ["FARGATE"],
    "cpu": "256",
    "memory": "512",
    "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
    "taskRoleArn": "arn:aws:iam::123456789012:role/myAppTaskRole",
    "containerDefinitions": [
        {
            "name": "my-app",
            "image": "123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest",
            "portMappings": [
                {
                    "containerPort": 8080,
                    "protocol": "tcp"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "/aws/ecs/my-app",
                    "awslogs-region": "us-west-2",
                    "awslogs-stream-prefix": "ecs"
                }
            },
            "secrets": [
                {
                    "name": "DATABASE_PASSWORD",
                    "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:myapp/db-password-AbCdEf"
                },
                {
                    "name": "API_KEY",
                    "valueFrom": "arn:aws:ssm:us-west-2:123456789012:parameter/myapp/api-key"
                }
            ],
            "environment": [
                {
                    "name": "APP_ENV",
                    "value": "production"
                }
            ]
        }
    ]
}
```

## Task Role

### Minimal Application Task Role (Example)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:GetObject"], "Resource": "arn:aws:s3:::my-app-bucket/*" },
    { "Effect": "Allow", "Action": ["dynamodb:Query","dynamodb:GetItem","dynamodb:PutItem"], "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/my-app-table" }
  ]
}
```

(Advanced conditional policies, KMS examples, and broad service samples removed for brevity.)

## Credential Injection Mechanisms

### ECS Credential Provider Endpoint

ECS automatically exposes a metadata endpoint that provides temporary credentials to containers.

#### Credential Endpoint Structure

### Environment Variables Injection

ECS automatically sets environment variables that the AWS SDK uses for credential discovery:

```bash
# ECS sets these automatically
AWS_CONTAINER_CREDENTIALS_RELATIVE_URI="/v2/credentials/12345678-1234-1234-1234-123456789012"
AWS_REGION="ap-southeast-6"

# Credentials endpoint
wget -qO- http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```

![ECS_ENV](/images/ecs-task-role.png)

#### Sample Credential Response

```json
{
    "RoleArn": "arn:aws:iam::123456789012:role/myAppTaskRole",
    "AccessKeyId": "ASIAIOSFODNN7EXAMPLE",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    "Token": "IQoJb3JpZ2luX2VjEHoaCXVzLXdlc3QtMiJIMEYCIQDSu...",
    "Expiration": "2025-08-19T14:30:00Z"
}
```

### Credential Injection Timeline

```
Container Startup Timeline:

1. ECS Agent receives task definition
   └── Uses Task Execution Role to:
       ├── Pull container image from ECR
       ├── Retrieve secrets from Parameter Store/Secrets Manager
       └── Set up CloudWatch logging

2. Container starts
   └── ECS injects environment variables:
       ├── AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
       └── AWS_DEFAULT_REGION

3. Application code runs
   └── AWS SDK discovers credentials via:
       ├── Environment variables
       ├── Makes HTTP call to metadata service
       └── Receives temporary credentials from Task Role

4. Credential refresh (automatic)
   └── AWS SDK automatically refreshes credentials before expiration
```

## AWS SDK Credential Discovery Process

The AWS SDK follows a specific order when looking for credentials:

```
AWS SDK Credential Chain (ECS Context):

1. Environment Variables
   ├── AWS_ACCESS_KEY_ID
   ├── AWS_SECRET_ACCESS_KEY
   └── AWS_SESSION_TOKEN

2. Java System Properties (Java SDK only)
   ├── aws.accessKeyId
   └── aws.secretKey

3. Credential profiles file
   └── ~/.aws/credentials

4. Container credentials
   └── AWS_CONTAINER_CREDENTIALS_RELATIVE_URI

5. Instance profile credentials (EC2 only)
   └── http://169.254.169.254/latest/meta-data/iam/security-credentials/

6. Web Identity Token (EKS/IRSA)
   └── AWS_WEB_IDENTITY_TOKEN_FILE
```

Key point: No credential wiring required—ECS provides them automatically.

## Conclusion

Understanding the distinction between ECS Task Execution Roles and Task Roles is crucial for building secure, efficient containerized applications on AWS.
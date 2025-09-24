---
title: "AWS Batch on Fargate: Single-Node, Multi-Container Jobs for Sidecar Pipelines"
date: 2025-08-22T00:00:01Z
draft: false
summary: "This post shows how to run single-node, multi-container jobs on AWS Batch with Fargate, and why that’s powerful for sidecar-style data pipelines."
tags: ['aws', 'batch', 'fargate', 'ecs', 'containers', 'serverless', 'data-pipeline', 'sidecar', 'orchestration']
---

This post shows how to run single-node, multi-container jobs on AWS Batch with Fargate, and why that’s powerful for sidecar-style data pipelines. We’ll build a pattern where a “main app” container exposes a local API, and a lightweight “pipeline” container consumes that API and exits when it’s done—letting the whole job complete cleanly without managing servers.

## Why multi-container on a single Batch node?

- Co-locate cooperating processes on the same task/node
- Low-latency localhost communication between containers
- Shared lifecycle and logs managed by Batch
- Simple job success criteria (e.g., pipeline container exit code)
- No EC2 instances to manage (Fargate)

Common use cases:
- Data extraction sidecar that scrapes an in-task API and writes to S3

## How it works (Batch + Fargate + ECS orchestration)

- AWS Batch now supports ECS-style multi-container jobs via the ECS orchestration mode
- A single Batch job runs one ECS task (single node) with multiple containers
- Containers share the task network namespace and optional shared volumes
- You control container start order with `dependsOn` and health checks
- Batch decides job success/failure based on container exit codes and `essential` flags

High-level flow:
1) Batch schedules a Fargate task
2) “Main app” container starts and becomes healthy (exposes `:8080/health`)
3) “Pipeline” container starts after main is healthy, calls `http://localhost:8080/api`, processes data, exits 0
4) Batch stops the task; job marked Succeeded if essential containers succeeded

## Job definition (single node, multiple containers)

Below is an example AWS Batch job definition JSON using ECS orchestration with Fargate. It starts `main-app` first and only starts `data-pipeline` after the main container is healthy. The pipeline exits to signal job completion.

Notes:
- Mark the pipeline container as `essential: true` so its exit code drives job success
- Mark the main container as `essential: false` (it’s just providing an API for the pipeline)
- Use a health check on the main container so `dependsOn: HEALTHY` works

```json
{
  "jobDefinitionName": "fargate-multicontainer-sidecar",
  "type": "container",
  "orchestrationType": "ECS",
  "platformCapabilities": ["FARGATE"],
  "ecsProperties": {
    "taskProperties": [
      {
        "networkConfiguration": {
          "assignPublicIp": "ENABLED"
        },
        "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
        "taskRoleArn": "arn:aws:iam::123456789012:role/myJobTaskRole",
        "runtimePlatform": {
          "cpuArchitecture": "X86_64",
          "operatingSystemFamily": "LINUX"
        },
        "fargatePlatformConfiguration": {
          "platformVersion": "LATEST"
        },
        "ephemeralStorage": {
          "sizeInGiB": 30
        },
        "cpu": "1024",
        "memory": "2048",
        "containers": [
          {
            "name": "main-app",
            "image": "123456789012.dkr.ecr.ap-southeast-6.amazonaws.com/main-app:latest",
            "essential": false,
            "portMappings": [
              { "containerPort": 8080, "hostPort": 8080, "protocol": "tcp" }
            ],
            "healthCheck": {
              "command": ["CMD-SHELL", "curl -fsS http://localhost:8080/health || exit 1"],
              "interval": 5,
              "timeout": 2,
              "retries": 10,
              "startPeriod": 5
            },
            "environment": [
              { "name": "SERVER_LOG", "value": "info" }
            ],
            "logConfiguration": {
              "logDriver": "awslogs",
              "options": {
                "awslogs-group": "/aws/batch/job/fargate-multicontainer-sidecar",
                "awslogs-region": "ap-southeast-6",
                "awslogs-stream-prefix": "main"
              }
            }
          },
          {
            "name": "data-pipeline",
            "image": "123456789012.dkr.ecr.ap-southeast-6.amazonaws.com/data-pipeline:latest",
            "essential": true,
            "dependsOn": [
              { "containerName": "main-app", "condition": "HEALTHY" }
            ],
            "environment": [
              { "name": "API_BASE_URL", "value": "http://localhost:8080" },
              { "name": "OUTPUT_BUCKET", "value": "s3://my-bucket/exports/" }
            ],
            "command": ["Ref::EVENT_JSON"], # Payload
            "logConfiguration": {
              "logDriver": "awslogs",
              "options": {
                "awslogs-group": "/aws/batch/job/fargate-multicontainer-sidecar",
                "awslogs-region": "ap-southeast-6",
                "awslogs-stream-prefix": "pipeline"
              }
            }
          }
        ]
      }
    ]
  },
  "retryStrategy": { "attempts": 1 },
  "propagateTags": true,
  "tags": { "project": "sidecar-pipeline" }
}
```

## Minimal container examples

Main app: simple HTTP API (Node.js example)

```javascript
// main-app/server.js
const express = require('express');
const app = express();

app.get('/health', (req, res) => res.send('ok'));

app.get('/api', (req, res) => {
  // Simulate data generation
  res.json({ items: [1, 2, 3], generatedAt: new Date().toISOString() });
});

const port = process.env.PORT || 8080;
app.listen(port, () => console.log(`main-app listening on ${port}`));
```

```dockerfile
# main-app/Dockerfile
FROM public.ecr.aws/docker/library/node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY server.js ./
EXPOSE 8080
CMD ["node", "server.js"]
```

Data pipeline: call local API and write to S3 (Python example)

```python
# data-pipeline/pipeline.py
import os, json, urllib.request, boto3

base = os.getenv('API_BASE_URL', 'http://localhost:8080')
output = os.getenv('OUTPUT_BUCKET')  # e.g. s3://bucket/prefix/

with urllib.request.urlopen(f"{base}/api") as r:
    body = r.read()
    data = json.loads(body)

print("Fetched:", data)

if output and output.startswith('s3://'):
    s3 = boto3.client('s3')
    _, rest = output.split('s3://', 1)
    bucket, prefix = rest.split('/', 1)
    key = f"{prefix.rstrip('/')}/export.json"
    s3.put_object(Bucket=bucket, Key=key, Body=json.dumps(data).encode('utf-8'))
    print(f"Wrote s3://{bucket}/{key}")
```

```dockerfile
# data-pipeline/Dockerfile
FROM public.ecr.aws/docker/library/python:3.12-alpine
RUN pip install boto3
WORKDIR /app
COPY pipeline.py ./
CMD ["python", "/app/pipeline.py"]
```

## IAM roles

- Task execution role: pull ECR images, write CloudWatch Logs, (optional) read Secrets Manager/SSM
- Task role (jobRoleArn): permissions for the pipeline (e.g., `s3:PutObject`)

Example policy attachment for pipeline writes:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:PutObjectAcl"],
      "Resource": "arn:aws:s3:::my-bucket/exports/*"
    }
  ]
}
```

## Submitting the job

- Register the job definition
- Create a Batch job queue and compute environment (Fargate/Fargate Spot)
- Submit the job referencing the job definition; optionally pass env overrides (e.g., S3 prefix)

Example (CLI, optional):

```bash
# Register job definition
aws batch register-job-definition \
  --cli-input-json file://jobdef-fargate-multicontainer.json

# Submit job
aws batch submit-job \
  --job-name export-run-$(date +%s) \
  --job-queue my-fargate-queue \
  --job-definition fargate-multicontainer-sidecar \
  --container-overrides '[{"name":"data-pipeline","environment":[{"name":"OUTPUT_BUCKET","value":"s3://my-bucket/exports/run-123/"}]}]'
```

## Behavior details and knobs

- Start order: use `dependsOn` + `healthCheck` to ensure the API is ready before the pipeline starts
- Job success: make the sidecar pipeline `essential: true`; when it exits 0, job succeeds; Batch will stop the task and non-essential containers
- Storage: use `ephemeralStorage` for temp space or define `volumes` + `mountPoints` for shared files
- Logging: send both containers to the same CloudWatch Logs group with different stream prefixes
- Cost: Fargate price is per-task vCPU/memory duration; keep the main container minimal (or non-essential) so the job ends quickly

## Takeaways

- Batch + Fargate multi-container jobs are perfect for short-lived, cooperating processes
- Use `dependsOn` and health checks to coordinate start-up
- Make the sidecar pipeline the `essential` container so its exit code controls job success
- Keep containers minimal and the job short to optimize cost

This pattern gives you a clean, serverless data pipeline that travels with the job—no separate services to provision, cost saving, and excellent observability through Batch and CloudWatch Logs.

# Opsera AWS Lambda Deployment Skill

## Description
Creates Opsera pipelines for AWS Lambda serverless function deployments with build automation, S3 artifact storage, and security scanning. Supports Java, Node.js, Python, and .NET runtimes.

## Usage
```
/opsera-aws-lambda [options]
```

## Parameters
- `--name`: Pipeline name (required)
- `--repo`: GitHub repository (org/repo format)
- `--runtime`: java | node | python | dotnet (default: java)
- `--build-tool`: maven | gradle | npm | pip | dotnet (auto-detected from runtime)
- `--function-name`: AWS Lambda function name
- `--s3-bucket`: S3 bucket for artifact storage
- `--aws-region`: AWS region (default: us-east-1)
- `--security-scan`: veracode | sonar | snyk | none (default: veracode)
- `--notifications`: slack | teams | email (comma-separated)

## Examples
```
/opsera-aws-lambda --name "My Lambda" --repo "org/lambda-func" --runtime java
/opsera-aws-lambda --name "API Lambda" --function-name "my-api-handler" --s3-bucket "my-lambda-artifacts"
/opsera-aws-lambda --name "Python Lambda" --runtime python --security-scan sonar
```

---

## Pipeline Architecture

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│  Build & Package    │───▶│  Push to S3         │───▶│  S3 Artifact Ref    │
│  (Maven/npm/pip)    │    │  (Upload)           │    │  (Download URL)     │
│  Creates artifact   │    │                     │    │                     │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
                                                                │
                                                                ▼
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│  Security Scan      │◀───│  Deploy to Lambda   │◀───│                     │
│  (Veracode/Sonar)   │    │  (Create/Update)    │    │                     │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

---

## Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

Generate unique IDs using pattern: `[6-char prefix][18-char random hex]`
Example: `677101a1b2c3d4e5f6a7b8c1`

---

## Tool Identifiers Reference

| Step Type | tool_identifier | Purpose |
|-----------|-----------------|---------|
| Build (Jenkins) | `jenkins` | Build application with Maven/Gradle/npm |
| S3 Upload | `s3` | Upload artifact to S3 bucket |
| S3 Reference | `s3` | Create S3 download URL reference |
| Lambda Deploy | `aws_lambda` | Create or update Lambda function |
| Veracode Scan | `veracode` | Security vulnerability scanning |
| Sonar Scan | `sonar` | Code quality analysis |

---

## Complete Pipeline Template

### Java Lambda Pipeline with Maven

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "AWS Lambda deployment pipeline with Maven build and security scanning",
  "type": ["serverless"],
  "tags": ["aws", "lambda", "serverless", "java", "maven"],
  "workflow": [
    {
      "_id": "677101a1b2c3d4e5f6a7b8c1",
      "name": "Maven Build",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "jenkins",
        "job_type": "opsera-job",
        "configuration": {
          "jobType": "BUILD",
          "buildType": "maven",
          "buildToolVersion": "6.3",
          "mavenTask": "clean install",
          "gitBranch": "main",
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[REPO_URL]",
          "repository": "[REPO_NAME]",
          "repoId": "[ORG/REPO]",
          "projectId": "[ORG/REPO]",
          "service": "github",
          "toolConfigId": "[JENKINS_TOOL_ID]",
          "agentLabels": "generic-linux",
          "autoScaleEnable": true,
          "workspaceDeleteFlag": true,
          "outputFileName": "*.jar",
          "outputPath": "target",
          "enableOutputVariables": true,
          "dependencies": {
            "java": "8",
            "maven": "3.8.2"
          },
          "dependencyType": [
            {
              "dependencyType": "java",
              "name": "Java openJDK Version 8",
              "version": "8"
            },
            {
              "dependencyType": "maven",
              "name": "Maven Version 3.8.2",
              "version": "3.8.2"
            }
          ],
          "customParameters": []
        }
      },
      "type": ["build"]
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c2",
      "name": "S3 Push",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "s3",
        "configuration": {
          "jobType": "SEND S3",
          "buildStepId": "677101a1b2c3d4e5f6a7b8c1",
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "bucketName": "[S3_BUCKET_NAME]",
          "bucketAccess": "private",
          "regions": "us-east-1"
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c3",
      "name": "S3 Artifact Reference",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "s3",
        "configuration": {
          "jobType": "SEND S3",
          "buildStepId": "677101a1b2c3d4e5f6a7b8c1",
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "bucketName": "[S3_BUCKET_NAME]",
          "bucketAccess": "private",
          "regions": "us-east-1"
        }
      },
      "type": ["deploy"]
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c4",
      "name": "Deploy to Lambda",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c3"],
      "tool": {
        "tool_identifier": "aws_lambda",
        "configuration": {
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "lambdaAction": "create",
          "lambdaTasks": []
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c5",
      "name": "Veracode Security Scan",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c4"],
      "tool": {
        "tool_identifier": "veracode",
        "configuration": {
          "buildStepId": "677101a1b2c3d4e5f6a7b8c1",
          "toolConfigId": "[VERACODE_TOOL_ID]",
          "clientSideThreshold": true,
          "thresholdVulnerability": [
            {
              "level": "5",
              "count": 0
            },
            {
              "level": "4",
              "count": 5
            }
          ]
        }
      },
      "type": ["compliance-sec"]
    }
  ]
}
```

---

## Runtime-Specific Build Configurations

### Java with Maven

```json
{
  "buildType": "maven",
  "buildToolVersion": "6.3",
  "mavenTask": "clean install",
  "outputFileName": "*.jar",
  "outputPath": "target",
  "dependencies": {
    "java": "8",
    "maven": "3.8.2"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 8", "version": "8"},
    {"dependencyType": "maven", "name": "Maven Version 3.8.2", "version": "3.8.2"}
  ]
}
```

### Java with Gradle

```json
{
  "buildType": "gradle",
  "buildToolVersion": "7.0",
  "gradleTask": "clean build",
  "outputFileName": "*.jar",
  "outputPath": "build/libs",
  "dependencies": {
    "java": "11",
    "gradle": "7.0"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 11", "version": "11"},
    {"dependencyType": "gradle", "name": "Gradle Version 7.0", "version": "7.0"}
  ]
}
```

### Node.js

```json
{
  "buildType": "npm",
  "outputFileName": "*.zip",
  "outputPath": ".",
  "commands": "npm ci\nnpm run build\nzip -r lambda-function.zip . -x 'node_modules/*' -x '.git/*'",
  "dependencies": {
    "node": "18"
  },
  "dependencyType": [
    {"dependencyType": "node", "name": "Node.js Version 18", "version": "18"}
  ]
}
```

### Python

```json
{
  "buildType": "python",
  "outputFileName": "lambda_function.zip",
  "outputPath": ".",
  "commands": "pip install -r requirements.txt -t ./package\ncp -r *.py package/\ncd package\nzip -r ../lambda_function.zip .",
  "dependencies": {
    "python": "3.9"
  },
  "dependencyType": [
    {"dependencyType": "python", "name": "Python Version 3.9", "version": "3.9"}
  ]
}
```

### .NET

```json
{
  "buildType": "dotnet",
  "outputFileName": "lambda-deployment.zip",
  "outputPath": "bin/Release/net6.0",
  "commands": "dotnet restore\ndotnet build --configuration Release\ndotnet publish --configuration Release --output ./publish\ncd publish\nzip -r ../lambda-deployment.zip .",
  "dependencies": {
    "dotnet": "6.0"
  },
  "dependencyType": [
    {"dependencyType": "dotnet", "name": ".NET SDK Version 6.0", "version": "6.0"}
  ]
}
```

---

## S3 Configuration

### S3 Upload Step

```json
{
  "_id": "677101a1b2c3d4e5f6a7b8c2",
  "name": "S3 Push",
  "active": true,
  "dependencies": ["[BUILD_STEP_ID]"],
  "tool": {
    "tool_identifier": "s3",
    "configuration": {
      "jobType": "SEND S3",
      "buildStepId": "[BUILD_STEP_ID]",
      "awsToolConfigId": "[AWS_TOOL_ID]",
      "bucketName": "[S3_BUCKET_NAME]",
      "bucketAccess": "private",
      "regions": "[AWS_REGION]"
    }
  },
  "type": ["publish"]
}
```

### S3 Artifact Reference Step

This step creates the S3 download URL that Lambda will use:

```json
{
  "_id": "677101a1b2c3d4e5f6a7b8c3",
  "name": "S3 Artifact Reference",
  "active": true,
  "dependencies": ["[S3_PUSH_STEP_ID]"],
  "tool": {
    "tool_identifier": "s3",
    "configuration": {
      "jobType": "SEND S3",
      "buildStepId": "[BUILD_STEP_ID]",
      "awsToolConfigId": "[AWS_TOOL_ID]",
      "bucketName": "[S3_BUCKET_NAME]",
      "bucketAccess": "private",
      "regions": "[AWS_REGION]",
      "artifactDownloadUrl": "s3://[BUCKET]/[PATH]/[ARTIFACT]",
      "artifactName": "[ARTIFACT_NAME]",
      "s3Url": "https://[BUCKET].s3.[REGION].amazonaws.com/[PATH]/[ARTIFACT]"
    }
  },
  "type": ["deploy"]
}
```

**Configuration Options:**
| Field | Description |
|-------|-------------|
| `buildStepId` | Reference to the build step that created the artifact |
| `awsToolConfigId` | AWS credentials configuration ID |
| `bucketName` | S3 bucket name for artifact storage |
| `bucketAccess` | `private` or `public` bucket access |
| `regions` | AWS region for S3 bucket |
| `artifactDownloadUrl` | S3 URI for Lambda deployment |
| `s3Url` | HTTPS URL to S3 artifact |

---

## Lambda Deployment Configuration

### Create or Update Lambda Function

```json
{
  "_id": "677101a1b2c3d4e5f6a7b8c4",
  "name": "Deploy to Lambda",
  "active": true,
  "dependencies": ["[S3_REFERENCE_STEP_ID]"],
  "tool": {
    "tool_identifier": "aws_lambda",
    "configuration": {
      "awsToolConfigId": "[AWS_TOOL_ID]",
      "lambdaAction": "create",
      "lambdaTasks": []
    }
  },
  "type": ["publish"]
}
```

**Lambda Actions:**
| Action | Description |
|--------|-------------|
| `create` | Create new Lambda function or update if exists |
| `update` | Update existing Lambda function code |
| `deploy` | Deploy function from S3 artifact |

**Configuration Options:**
| Field | Description |
|-------|-------------|
| `awsToolConfigId` | AWS credentials configuration ID |
| `lambdaAction` | Action to perform (create/update/deploy) |
| `lambdaTasks` | Array of Lambda function configurations |

---

## Security Scanning Options

### Veracode Scanning

```json
{
  "_id": "677101a1b2c3d4e5f6a7b8c5",
  "name": "Veracode Security Scan",
  "active": true,
  "dependencies": ["[LAMBDA_DEPLOY_STEP_ID]"],
  "tool": {
    "tool_identifier": "veracode",
    "configuration": {
      "buildStepId": "[BUILD_STEP_ID]",
      "toolConfigId": "[VERACODE_TOOL_ID]",
      "clientSideThreshold": true,
      "thresholdVulnerability": [
        {
          "level": "5",
          "count": 0
        },
        {
          "level": "4",
          "count": 5
        },
        {
          "level": "3",
          "count": 10
        }
      ]
    }
  },
  "type": ["compliance-sec"]
}
```

**Veracode Severity Levels:**
- Level 5: Very High (Critical)
- Level 4: High
- Level 3: Medium
- Level 2: Low
- Level 1: Very Low

### Sonar Code Quality

```json
{
  "_id": "677101a1b2c3d4e5f6a7b8c5",
  "name": "Sonar Code Quality",
  "active": true,
  "dependencies": ["[LAMBDA_DEPLOY_STEP_ID]"],
  "tool": {
    "tool_identifier": "sonar",
    "job_type": "opsera-job",
    "configuration": {
      "jobType": "CODE SCAN",
      "buildToolVersion": "6.3",
      "buildType": "maven",
      "gitBranch": "main",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "repository": "[REPO_NAME]",
      "projectKey": "[SONAR_PROJECT_KEY]",
      "projectName": "[SONAR_PROJECT_NAME]",
      "sonarToolConfigId": "[SONAR_TOOL_ID]",
      "toolConfigId": "[JENKINS_TOOL_ID]",
      "agentLabels": "generic-linux",
      "executionEngine": "jenkins"
    }
  },
  "type": ["code-scan"]
}
```

---

## Extended Pipeline with Pre/Post Steps

### Full Pipeline with Validation and Testing

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "AWS Lambda deployment with validation, security scanning, and testing",
  "type": ["serverless"],
  "tags": ["aws", "lambda", "serverless"],
  "workflow": [
    {
      "_id": "677101a1b2c3d4e5f6a7b8c0",
      "name": "Pre-Build Validation",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "PreBuildValidation",
          "script": "#!/bin/bash\necho \"Validating Lambda project structure...\"\nif [ -f \"pom.xml\" ] || [ -f \"build.gradle\" ] || [ -f \"package.json\" ]; then\n  echo \"Build file found\"\nelse\n  echo \"ERROR: No build file found\"\n  exit 1\nfi\necho \"Validation complete\"",
          "environmentVariables": [],
          "plugins": [
            {
              "name": "Git",
              "plugin": "git_repository",
              "configuration": {
                "branch": "main",
                "repository": "[REPO_NAME]",
                "repositoryId": "[ORG/REPO]",
                "repositoryUrl": "[REPO_URL]",
                "repositoryToolId": "[GIT_TOOL_ID]",
                "repositoryToolIdentifier": "github"
              }
            }
          ]
        }
      },
      "type": ["script"]
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c1",
      "name": "Build",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c0"],
      "tool": {"tool_identifier": "jenkins"}
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c2",
      "name": "S3 Push",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c1"],
      "tool": {"tool_identifier": "s3"}
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c3",
      "name": "S3 Reference",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c2"],
      "tool": {"tool_identifier": "s3"}
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c4",
      "name": "Deploy to Lambda",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c3"],
      "tool": {"tool_identifier": "aws_lambda"}
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c5",
      "name": "Security Scan",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c4"],
      "tool": {"tool_identifier": "veracode"}
    },
    {
      "_id": "677101a1b2c3d4e5f6a7b8c6",
      "name": "Lambda Smoke Test",
      "active": true,
      "dependencies": ["677101a1b2c3d4e5f6a7b8c4"],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "LambdaSmokeTest",
          "script": "#!/bin/bash\nset -e\necho \"=== Testing Lambda Function ===\"\nFUNCTION_NAME=\"[LAMBDA_FUNCTION_NAME]\"\nREGION=\"us-east-1\"\n\n# Invoke Lambda function\necho \"Invoking Lambda function: $FUNCTION_NAME\"\nRESPONSE=$(aws lambda invoke --function-name $FUNCTION_NAME --region $REGION --payload '{}' response.json 2>&1)\n\nif echo $RESPONSE | grep -q \"StatusCode: 200\"; then\n  echo \"Lambda invocation successful\"\n  cat response.json\nelse\n  echo \"Lambda invocation failed\"\n  echo $RESPONSE\n  exit 1\nfi\n\necho \"=== Lambda Test Passed ===\"",
          "environmentVariables": [],
          "secretEnvironmentVariables": []
        }
      },
      "type": ["test"]
    }
  ]
}
```

---

## Notification Configuration

```json
{
  "notification": [
    {
      "type": "slack",
      "enabled": true,
      "event": "all",
      "logEnabled": true,
      "channel": "[SLACK_CHANNEL]",
      "toolId": "[SLACK_TOOL_ID]"
    },
    {
      "type": "teams",
      "enabled": true,
      "event": "all",
      "logEnabled": true,
      "teamsType": "webhook",
      "toolId": "[TEAMS_TOOL_ID]"
    },
    {
      "type": "email",
      "enabled": true,
      "event": "error",
      "addresses": ["team@company.com"]
    }
  ]
}
```

---

## API Response Structure

On successful pipeline creation:

```json
{
  "status": "success",
  "data": {
    "pipelineId": "67581ce81a1cda01b6f3f8ee",
    "pipeline": {
      "_id": "67581ce81a1cda01b6f3f8ee",
      "name": "NC-aws lambda",
      "active": true,
      "type": ["serverless"],
      "workflow": {
        "run_count": 0,
        "plan": [...]
      }
    }
  }
}
```

---

## Implementation Instructions

### 1. Gather Requirements
- Pipeline name
- Source repository (GitHub/GitLab/Bitbucket)
- Runtime (Java, Node.js, Python, .NET)
- Lambda function name
- S3 bucket for artifact storage
- AWS region
- Security scanning preference

### 2. Create Pipeline via API

```
mcp__opsera-ai-agent__post_api_v2_pipeline_create
```

### 3. Required Tool IDs
Before creating the pipeline, you need:
- `gitToolId`: Git tool configuration ID
- `toolConfigId`: Jenkins tool configuration ID
- `awsToolConfigId`: AWS credentials ID
- `veracodeToolId`: Veracode configuration ID (if using Veracode)
- `sonarToolConfigId`: Sonar configuration ID (if using Sonar)

---

## Best Practices

1. **Use private S3 buckets** for artifact storage
2. **Enable versioning** on S3 bucket for rollback capability
3. **Configure Lambda timeout** appropriately for your function
4. **Set memory limits** based on function requirements
5. **Use environment variables** for configuration
6. **Enable CloudWatch logging** for Lambda functions
7. **Configure VPC** if Lambda needs private resource access
8. **Add security scanning** before production deployment
9. **Use separate Lambda functions** for different environments
10. **Implement proper IAM roles** with least privilege

---

## Lambda Function Configuration

### Environment Variables

Pass configuration to Lambda via environment variables:

```json
{
  "environmentVariables": [
    {"name": "ENVIRONMENT", "value": "production"},
    {"name": "LOG_LEVEL", "value": "INFO"},
    {"name": "API_ENDPOINT", "value": "https://api.example.com"}
  ]
}
```

### Runtime Configuration

| Runtime | Handler Format | Example |
|---------|----------------|---------|
| Java | `package.Class::method` | `com.example.Handler::handleRequest` |
| Node.js | `file.function` | `index.handler` |
| Python | `file.function` | `lambda_function.lambda_handler` |
| .NET | `Assembly::Namespace.Class::Method` | `LambdaFunction::Handler.Function::FunctionHandler` |

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `677101a1b2c3d4e5f6a7b8c1`

2. **"Missing buildStepId reference"**
   - Cause: S3 or Lambda step can't find build step
   - Fix: Ensure `buildStepId` matches the build step's `_id`

### Runtime Errors

1. **AWS authentication failure**: Verify AWS credentials are valid
2. **S3 bucket not found**: Check bucket name and region
3. **Lambda deployment failed**: Verify IAM role and permissions
4. **Build artifact not found**: Check output path and file name
5. **Security scan timeout**: Increase timeout or optimize scan scope

### Lambda Specific Issues

1. **Function timeout**: Increase Lambda timeout in configuration
2. **Memory limit exceeded**: Increase Lambda memory allocation
3. **VPC connectivity**: Check security groups and subnet configuration
4. **Cold start latency**: Consider provisioned concurrency
5. **Package size limit**: Optimize dependencies or use Lambda layers

---

## Reference Pipeline

See pipeline `67581ce81a1cda01b6f3f8ee` (NC-aws lambda) for a complete working example with 17 successful runs.

---

## Additional Resources

### AWS Lambda Limits
- Deployment package size: 50 MB (zipped), 250 MB (unzipped)
- `/tmp` directory storage: 512 MB
- Timeout: Max 15 minutes
- Memory: 128 MB to 10,240 MB
- Concurrent executions: 1,000 (soft limit)

### Recommended Lambda Layers
- AWS SDK layer (reduces package size)
- Custom dependencies layer
- Common utility functions layer
- Monitoring and observability layer

### Cost Optimization
- Use ARM-based Graviton2 processors for up to 34% cost savings
- Optimize memory allocation (affects CPU allocation)
- Minimize cold starts with provisioned concurrency
- Use S3 lifecycle policies to archive old artifacts

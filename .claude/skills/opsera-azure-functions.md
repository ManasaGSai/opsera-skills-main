# Opsera Azure Functions Deployment Skill

> **Branding**: Follow the Opsera branding guidelines in `.claude/CLAUDE.md` for all responses.

## Description
Creates Opsera pipelines for Azure Functions deployments using ZIP deployment method. Supports Java, Node.js, Python, and .NET function apps.

## Usage
```
/opsera-azure-functions [options]
```

## Parameters
- `--name`: Pipeline name (required)
- `--repo`: GitHub repository (org/repo format)
- `--runtime`: java | node | python | dotnet (default: java)
- `--function-app`: Azure Function App name
- `--resource-group`: Azure resource group name
- `--storage-account`: Azure storage account name
- `--container`: Blob container name for artifacts
- `--notifications`: slack | teams | email (comma-separated)

## Examples
```
/opsera-azure-functions --name "My Function" --repo "org/azure-func" --runtime java
/opsera-azure-functions --name "API Functions" --function-app "myapp-func" --resource-group "myapp-rg"
```

---

## Pipeline Architecture

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│  Build & Package    │───▶│  Push to Azure      │───▶│  Deploy Azure       │
│  (Maven/npm/pip)    │    │  Blob Storage       │    │  Functions          │
│  Creates ZIP        │    │                     │    │  (ZIP Deploy)       │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

---

## Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

Generate unique IDs using pattern: `[6-char prefix][18-char random hex]`
Example: `676901a1b2c3d4e5f6a7b8c1`

---

## Tool Identifiers Reference

| Step Type | tool_identifier | Purpose |
|-----------|-----------------|---------|
| Build script | `command-line` | Build and package application |
| Azure storage push | `azure-zip-deployment` | Upload artifact to blob storage |
| Azure Functions deploy | `azure-functions` | Deploy function app via ZIP |

---

## Complete Pipeline Template

### Java Azure Functions Pipeline

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Azure Functions ZIP deployment pipeline",
  "type": ["serverless"],
  "tags": ["azure", "functions", "serverless", "zip-deploy"],
  "workflow": [
    {
      "_id": "676901a1b2c3d4e5f6a7b8c1",
      "name": "Build and Package",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "command-line",
        "job_type": "shell script",
        "configuration": {
          "jobType": "SHELL SCRIPT",
          "buildType": "others",
          "agentLabels": "generic-linux",
          "autoScaleEnable": true,
          "workspaceDeleteFlag": true,
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[REPO_URL]",
          "gitBranch": "main",
          "repository": "[REPO_NAME]",
          "repoId": "[ORG/REPO]",
          "service": "github",
          "toolConfigId": "[JENKINS_TOOL_ID]",
          "commands": "mvn clean install\ncp target/*.jar function/\ncd function\nls\nzip -r archive.zip .\nls",
          "outputFileName": "archive.zip",
          "outputPath": "function",
          "dependencies": {
            "java": "8",
            "maven": "3.6.2"
          },
          "dependencyType": [
            {
              "dependencyType": "java",
              "name": "Java openJDK Version 8",
              "version": "8"
            },
            {
              "dependencyType": "maven",
              "name": "Maven Version 3.6.2",
              "version": "3.6.2"
            }
          ],
          "environmentVariables": [],
          "customParameters": []
        }
      },
      "type": ["script"]
    },
    {
      "_id": "676901a1b2c3d4e5f6a7b8c2",
      "name": "Push to Azure Storage",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "azure-zip-deployment",
        "configuration": {
          "buildStepId": "676901a1b2c3d4e5f6a7b8c1",
          "azureToolId": "[AZURE_TOOL_ID]",
          "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
          "azureStorageAccountName": "[STORAGE_ACCOUNT_NAME]",
          "resourceGroup": "[RESOURCE_GROUP]",
          "containerName": "[CONTAINER_NAME]",
          "containerPath": "",
          "artifactName": "archive.zip",
          "existingContainer": true,
          "useRunCount": true,
          "artifactVersion": false
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "676901a1b2c3d4e5f6a7b8c3",
      "name": "Deploy Azure Functions",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "azure-functions",
        "configuration": {
          "artifactStepId": "676901a1b2c3d4e5f6a7b8c2",
          "azureToolConfigId": "[AZURE_TOOL_ID]",
          "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
          "azureFunctionName": "[FUNCTION_APP_NAME]",
          "resourceGroupName": "[RESOURCE_GROUP]",
          "deploymentType": "zip",
          "existingFunctionName": true
        }
      },
      "type": ["publish"]
    }
  ]
}
```

---

## Runtime-Specific Build Commands

### Java (Maven)

```json
{
  "commands": "mvn clean install\ncp target/*.jar function/\ncd function\nzip -r archive.zip .\nls",
  "dependencies": {
    "java": "8",
    "maven": "3.6.2"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 8", "version": "8"},
    {"dependencyType": "maven", "name": "Maven Version 3.6.2", "version": "3.6.2"}
  ],
  "outputFileName": "archive.zip",
  "outputPath": "function"
}
```

### Java (Gradle)

```json
{
  "commands": "gradle clean build\ncp build/libs/*.jar function/\ncd function\nzip -r archive.zip .\nls",
  "dependencies": {
    "java": "11",
    "gradle": "7.0"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 11", "version": "11"},
    {"dependencyType": "gradle", "name": "Gradle Version 7.0", "version": "7.0"}
  ],
  "outputFileName": "archive.zip",
  "outputPath": "function"
}
```

### Node.js

```json
{
  "commands": "npm ci\nnpm run build\nzip -r archive.zip . -x 'node_modules/*' -x '.git/*'\nls -la",
  "dependencies": {
    "node": "18"
  },
  "dependencyType": [
    {"dependencyType": "node", "name": "Node.js Version 18", "version": "18"}
  ],
  "outputFileName": "archive.zip",
  "outputPath": "."
}
```

### Python

```json
{
  "commands": "pip install -r requirements.txt -t .python_packages/lib/site-packages\nzip -r archive.zip . -x '.git/*' -x '__pycache__/*'\nls -la",
  "dependencies": {
    "python": "3.9"
  },
  "dependencyType": [
    {"dependencyType": "python", "name": "Python Version 3.9", "version": "3.9"}
  ],
  "outputFileName": "archive.zip",
  "outputPath": "."
}
```

### .NET

```json
{
  "commands": "dotnet restore\ndotnet build --configuration Release\ndotnet publish --configuration Release --output ./publish\ncd publish\nzip -r ../archive.zip .\nls -la",
  "dependencies": {
    "dotnet": "6.0"
  },
  "dependencyType": [
    {"dependencyType": "dotnet", "name": ".NET SDK Version 6.0", "version": "6.0"}
  ],
  "outputFileName": "archive.zip",
  "outputPath": "."
}
```

---

## Azure Storage Configuration

### Push to Azure Blob Storage

```json
{
  "_id": "676901a1b2c3d4e5f6a7b8c2",
  "name": "Push to Azure Storage",
  "active": true,
  "dependencies": ["676901a1b2c3d4e5f6a7b8c1"],
  "tool": {
    "tool_identifier": "azure-zip-deployment",
    "configuration": {
      "buildStepId": "[BUILD_STEP_ID]",
      "azureToolId": "[AZURE_TOOL_ID]",
      "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
      "azureStorageAccountName": "[STORAGE_ACCOUNT]",
      "resourceGroup": "[RESOURCE_GROUP]",
      "containerName": "function-releases",
      "containerPath": "",
      "artifactName": "archive.zip",
      "existingContainer": true,
      "useRunCount": true,
      "artifactVersion": false
    }
  },
  "type": ["publish"]
}
```

**Configuration Options:**
| Field | Description |
|-------|-------------|
| `buildStepId` | Reference to the build step that creates the artifact |
| `azureToolId` | Azure tool configuration ID |
| `azureCredentialId` | Azure credentials ID |
| `azureStorageAccountName` | Azure storage account name |
| `resourceGroup` | Azure resource group |
| `containerName` | Blob container name |
| `containerPath` | Optional path within container |
| `artifactName` | Name of artifact file (e.g., archive.zip) |
| `existingContainer` | true if container already exists |
| `useRunCount` | Append run count to artifact name for versioning |

---

## Azure Functions Deployment Configuration

### ZIP Deployment

```json
{
  "_id": "676901a1b2c3d4e5f6a7b8c3",
  "name": "Deploy Azure Functions",
  "active": true,
  "dependencies": ["676901a1b2c3d4e5f6a7b8c2"],
  "tool": {
    "tool_identifier": "azure-functions",
    "configuration": {
      "artifactStepId": "[STORAGE_STEP_ID]",
      "azureToolConfigId": "[AZURE_TOOL_ID]",
      "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
      "azureFunctionName": "[FUNCTION_APP_NAME]",
      "resourceGroupName": "[RESOURCE_GROUP]",
      "deploymentType": "zip",
      "existingFunctionName": true,
      "applicationType": "",
      "azureRegion": "",
      "dynamicServiceName": "",
      "machine_type": "",
      "namePretext": "",
      "useCustomResourceGroup": ""
    }
  },
  "type": ["publish"]
}
```

**Configuration Options:**
| Field | Description |
|-------|-------------|
| `artifactStepId` | Reference to storage push step |
| `azureToolConfigId` | Azure tool configuration ID |
| `azureCredentialId` | Azure credentials ID |
| `azureFunctionName` | Name of the Azure Function App |
| `resourceGroupName` | Azure resource group |
| `deploymentType` | `zip` for ZIP deployment |
| `existingFunctionName` | true if Function App already exists |

---

## Extended Pipeline with Pre/Post Steps

### Full Pipeline with Validation and Testing

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Azure Functions deployment with validation and testing",
  "type": ["serverless"],
  "tags": ["azure", "functions", "serverless"],
  "workflow": [
    {
      "_id": "676901a1b2c3d4e5f6a7b8c0",
      "name": "Pre-Build Validation",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "PreBuildValidation",
          "script": "#!/bin/bash\necho \"Validating Azure Functions project structure...\"\nif [ -f \"host.json\" ]; then\n  echo \"host.json found\"\nelse\n  echo \"ERROR: host.json not found\"\n  exit 1\nfi\nif [ -f \"local.settings.json\" ] || [ -f \"local.settings.sample.json\" ]; then\n  echo \"Settings file found\"\nelse\n  echo \"WARNING: No settings file found\"\nfi\necho \"Validation complete\"",
          "environmentVariables": [],
          "secretEnvironmentVariables": [],
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
      "_id": "676901a1b2c3d4e5f6a7b8c1",
      "name": "Build and Package",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c0"],
      "tool": {
        "tool_identifier": "command-line",
        "job_type": "shell script",
        "configuration": {
          "jobType": "SHELL SCRIPT",
          "buildType": "others",
          "commands": "mvn clean install\ncp target/*.jar function/\ncd function\nzip -r archive.zip .",
          "outputFileName": "archive.zip",
          "outputPath": "function"
        }
      },
      "type": ["script"]
    },
    {
      "_id": "676901a1b2c3d4e5f6a7b8c2",
      "name": "Push to Azure Storage",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "azure-zip-deployment",
        "configuration": {
          "buildStepId": "676901a1b2c3d4e5f6a7b8c1",
          "containerName": "function-releases",
          "artifactName": "archive.zip",
          "useRunCount": true
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "676901a1b2c3d4e5f6a7b8c3",
      "name": "Deploy Azure Functions",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "azure-functions",
        "configuration": {
          "artifactStepId": "676901a1b2c3d4e5f6a7b8c2",
          "deploymentType": "zip",
          "existingFunctionName": true
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "676901a1b2c3d4e5f6a7b8c4",
      "name": "Post-Deploy Verification",
      "active": true,
      "dependencies": ["676901a1b2c3d4e5f6a7b8c3"],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "PostDeployVerification",
          "script": "#!/bin/bash\necho \"Verifying Azure Functions deployment...\"\nFUNCTION_URL=\"https://[FUNCTION_APP_NAME].azurewebsites.net/api/health\"\necho \"Checking endpoint: $FUNCTION_URL\"\nsleep 30\nHTTP_CODE=$(curl -s -o /dev/null -w \"%{http_code}\" $FUNCTION_URL)\nif [ \"$HTTP_CODE\" -eq 200 ]; then\n  echo \"Function app is responding (HTTP $HTTP_CODE)\"\nelse\n  echo \"WARNING: Function app returned HTTP $HTTP_CODE\"\nfi\necho \"Deployment verification complete\"",
          "environmentVariables": [],
          "secretEnvironmentVariables": []
        }
      },
      "type": ["script"]
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
    "pipelineId": "69391759c3674e21128f11e9",
    "pipeline": {
      "_id": "69391759c3674e21128f11e9",
      "name": "Golden AKSAzureFunctionsZIPdeployment",
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

## Opsera Portal URL

After creating a pipeline, provide the user with the direct link to view it in the Opsera portal:

```
https://portal.opsera.io/workflow/details/{pipelineId}/summary
```

**Example:**
```
Pipeline ID: 69391759c3674e21128f11e9
Portal URL: https://portal.opsera.io/workflow/details/69391759c3674e21128f11e9/summary
```

Always include this URL in the response after successful pipeline creation.

---

## Implementation Instructions

### 1. Gather Requirements
- Pipeline name
- Source repository (GitHub/GitLab/Bitbucket)
- Runtime (Java, Node.js, Python, .NET)
- Azure Function App name
- Azure resource group
- Azure storage account and container
- Build dependencies and versions

### 2. Create Pipeline via API

```
mcp__opsera-ai-agent__post_api_v2_pipeline_create
```

### 3. Required Tool IDs
Before creating the pipeline, you need:
- `gitToolId`: Git tool configuration ID
- `toolConfigId`: Jenkins tool configuration ID
- `azureToolId`: Azure tool configuration ID
- `azureCredentialId`: Azure credentials ID

---

## Best Practices

1. **Use ZIP deployment** for faster cold starts
2. **Enable useRunCount** for artifact versioning
3. **Validate project structure** before building
4. **Add post-deployment health checks** to verify function app
5. **Use separate storage containers** for different environments
6. **Configure appropriate runtime versions** matching Azure Function runtime
7. **Clean workspace after build** to prevent stale artifacts

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `676901a1b2c3d4e5f6a7b8c1`

2. **"Missing buildStepId reference"**
   - Cause: Storage push step can't find build step
   - Fix: Ensure `buildStepId` matches the build step's `_id`

3. **"Missing artifactStepId reference"**
   - Cause: Deploy step can't find storage step
   - Fix: Ensure `artifactStepId` matches the storage step's `_id`

### Runtime Errors

1. **Azure authentication failure**: Verify Azure credentials are valid
2. **Storage container not found**: Check `existingContainer` setting or create container
3. **Function app not found**: Verify function app name and resource group
4. **ZIP deployment failed**: Check artifact format and function app configuration
5. **Build dependencies missing**: Verify `dependencyType` configuration

---

## Reference Pipeline

See pipeline `69391759c3674e21128f11e9` (Golden AKSAzureFunctionsZIPdeployment) for a complete working example.

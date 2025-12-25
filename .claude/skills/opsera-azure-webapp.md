# Opsera Azure Web App Deployment Skill

> **Branding**: Follow the Opsera branding guidelines in `.claude/CLAUDE.md` for all responses.

## Description
Creates Opsera pipelines for Azure Web App deployments using package (ZIP) deployment. Supports Node.js, Java, Python, .NET, and other runtimes.

## Usage
```
/opsera-azure-webapp [options]
```

## Parameters
- `--name`: Pipeline name (required)
- `--repo`: GitHub repository (org/repo format)
- `--runtime`: node | java | python | dotnet (default: node)
- `--webapp-name`: Azure Web App name
- `--resource-group`: Azure resource group name
- `--storage-account`: Azure storage account name
- `--container`: Blob container name for artifacts
- `--notifications`: slack | teams | email (comma-separated)

## Examples
```
/opsera-azure-webapp --name "My Web App" --repo "org/webapp" --runtime node
/opsera-azure-webapp --name "API Service" --webapp-name "myapi-app" --resource-group "myapp-rg"
```

---

## Pipeline Architecture

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│  Build & Package    │───▶│  Push to Azure      │───▶│  Deploy to Azure    │
│  (npm/maven/pip)    │    │  Blob Storage       │    │  Web App            │
│  Creates ZIP        │    │                     │    │  (Package Deploy)   │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

---

## Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

Generate unique IDs using pattern: `[6-char prefix][18-char random hex]`
Example: `677001a1b2c3d4e5f6a7b8c1`

---

## Tool Identifiers Reference

| Step Type | tool_identifier | Purpose |
|-----------|-----------------|---------|
| Build script | `command-line` | Build and package application |
| Azure storage push | `azure-zip-deployment` | Upload artifact to blob storage |
| Azure Web App deploy | `azure_webapps` | Deploy to Azure Web App |

---

## Complete Pipeline Template

### Node.js Azure Web App Pipeline

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Azure Web App package deployment pipeline",
  "type": ["sdlc"],
  "tags": ["azure", "webapp", "nodejs", "package-deploy"],
  "workflow": [
    {
      "_id": "677001a1b2c3d4e5f6a7b8c1",
      "name": "Node.js Build",
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
          "workspaceDeleteFlag": false,
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[REPO_URL]",
          "gitBranch": "main",
          "repository": "[REPO_NAME]",
          "repoId": "[ORG/REPO]",
          "service": "github",
          "toolConfigId": "[JENKINS_TOOL_ID]",
          "commands": "apt update\napt install zip unzip\nnpm install\nzip -r nodeArtifact.zip .\nls -lrt",
          "outputFileName": "nodeArtifact.zip",
          "outputPath": "",
          "dependencies": {
            "node": "14"
          },
          "dependencyType": [
            {
              "dependencyType": "node",
              "name": "Node Version 14",
              "version": "14"
            }
          ],
          "environmentVariables": [],
          "customParameters": []
        }
      },
      "type": ["script"]
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c2",
      "name": "Azure Storage Push",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "azure-zip-deployment",
        "configuration": {
          "buildStepId": "677001a1b2c3d4e5f6a7b8c1",
          "azureToolId": "[AZURE_TOOL_ID]",
          "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
          "azureStorageAccountName": "[STORAGE_ACCOUNT_NAME]",
          "resourceGroup": "[RESOURCE_GROUP]",
          "containerName": "[CONTAINER_NAME]",
          "containerPath": "",
          "artifactName": "nodeArtifact.zip",
          "existingContainer": true,
          "useRunCount": false,
          "artifactVersion": false
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c3",
      "name": "Azure Web App Deploy",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "azure_webapps",
        "configuration": {
          "artifactStepId": "677001a1b2c3d4e5f6a7b8c2",
          "azureToolConfigId": "[AZURE_TOOL_ID]",
          "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
          "webappName": "[WEBAPP_NAME]",
          "resourceGroupName": "[RESOURCE_GROUP]",
          "deploymentType": "package",
          "webappPackageType": "zip",
          "webappCleanTargetPath": true,
          "webappTargetPath": "",
          "rollbackEnabled": false,
          "appSettingsEnabled": false,
          "appSettings": [],
          "connectionStringEnabled": false,
          "connectionStrings": []
        }
      },
      "type": ["deploy"]
    }
  ]
}
```

---

## Runtime-Specific Build Commands

### Node.js

```json
{
  "commands": "apt update\napt install zip unzip\nnpm install\nzip -r nodeArtifact.zip .\nls -lrt",
  "dependencies": {
    "node": "14"
  },
  "dependencyType": [
    {"dependencyType": "node", "name": "Node Version 14", "version": "14"}
  ],
  "outputFileName": "nodeArtifact.zip",
  "outputPath": ""
}
```

### Node.js (with Build Step)

```json
{
  "commands": "apt update\napt install zip unzip\nnpm ci\nnpm run build\nzip -r nodeArtifact.zip . -x 'node_modules/*' -x '.git/*'\nls -lrt",
  "dependencies": {
    "node": "18"
  },
  "dependencyType": [
    {"dependencyType": "node", "name": "Node Version 18", "version": "18"}
  ],
  "outputFileName": "nodeArtifact.zip",
  "outputPath": ""
}
```

### Java (Maven)

```json
{
  "commands": "mvn clean package -DskipTests\nmkdir -p deploy\ncp target/*.jar deploy/\ncd deploy\nzip -r ../javaArtifact.zip .\nls -lrt",
  "dependencies": {
    "java": "11",
    "maven": "3.8.6"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 11", "version": "11"},
    {"dependencyType": "maven", "name": "Maven Version 3.8.6", "version": "3.8.6"}
  ],
  "outputFileName": "javaArtifact.zip",
  "outputPath": ""
}
```

### Java (Gradle)

```json
{
  "commands": "gradle clean build -x test\nmkdir -p deploy\ncp build/libs/*.jar deploy/\ncd deploy\nzip -r ../javaArtifact.zip .\nls -lrt",
  "dependencies": {
    "java": "11",
    "gradle": "7.0"
  },
  "dependencyType": [
    {"dependencyType": "java", "name": "Java openJDK Version 11", "version": "11"},
    {"dependencyType": "gradle", "name": "Gradle Version 7.0", "version": "7.0"}
  ],
  "outputFileName": "javaArtifact.zip",
  "outputPath": ""
}
```

### Python

```json
{
  "commands": "apt update\napt install zip unzip python3-pip -y\npip install -r requirements.txt\nzip -r pythonArtifact.zip . -x '.git/*' -x '__pycache__/*' -x '*.pyc'\nls -lrt",
  "dependencies": {
    "python": "3.9"
  },
  "dependencyType": [
    {"dependencyType": "python", "name": "Python Version 3.9", "version": "3.9"}
  ],
  "outputFileName": "pythonArtifact.zip",
  "outputPath": ""
}
```

### .NET

```json
{
  "commands": "dotnet restore\ndotnet build --configuration Release\ndotnet publish --configuration Release --output ./publish\ncd publish\nzip -r ../dotnetArtifact.zip .\nls -lrt",
  "dependencies": {
    "dotnet": "6.0"
  },
  "dependencyType": [
    {"dependencyType": "dotnet", "name": ".NET SDK Version 6.0", "version": "6.0"}
  ],
  "outputFileName": "dotnetArtifact.zip",
  "outputPath": ""
}
```

---

## Azure Storage Configuration

### Push to Azure Blob Storage

```json
{
  "_id": "677001a1b2c3d4e5f6a7b8c2",
  "name": "Azure Storage Push",
  "active": true,
  "dependencies": ["677001a1b2c3d4e5f6a7b8c1"],
  "tool": {
    "tool_identifier": "azure-zip-deployment",
    "configuration": {
      "buildStepId": "[BUILD_STEP_ID]",
      "azureToolId": "[AZURE_TOOL_ID]",
      "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
      "azureStorageAccountName": "[STORAGE_ACCOUNT]",
      "resourceGroup": "[RESOURCE_GROUP]",
      "containerName": "webapp-storage-package",
      "containerPath": "",
      "artifactName": "nodeArtifact.zip",
      "existingContainer": true,
      "useRunCount": false,
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
| `artifactName` | Name of artifact file (e.g., nodeArtifact.zip) |
| `existingContainer` | true if container already exists |
| `useRunCount` | Append run count to artifact name for versioning |

---

## Azure Web App Deployment Configuration

### Package Deployment

```json
{
  "_id": "677001a1b2c3d4e5f6a7b8c3",
  "name": "Azure Web App Deploy",
  "active": true,
  "dependencies": ["677001a1b2c3d4e5f6a7b8c2"],
  "tool": {
    "tool_identifier": "azure_webapps",
    "configuration": {
      "artifactStepId": "[STORAGE_STEP_ID]",
      "azureToolConfigId": "[AZURE_TOOL_ID]",
      "azureCredentialId": "[AZURE_CREDENTIAL_ID]",
      "webappName": "[WEBAPP_NAME]",
      "resourceGroupName": "[RESOURCE_GROUP]",
      "deploymentType": "package",
      "webappPackageType": "zip",
      "webappCleanTargetPath": true,
      "webappTargetPath": "",
      "rollbackEnabled": false,
      "rollbackVersion": "",
      "appSettingsEnabled": false,
      "appSettings": [],
      "connectionStringEnabled": false,
      "connectionStrings": [],
      "azureRegion": "",
      "machine_type": ""
    }
  },
  "type": ["deploy"]
}
```

**Configuration Options:**
| Field | Description |
|-------|-------------|
| `artifactStepId` | Reference to storage push step |
| `azureToolConfigId` | Azure tool configuration ID |
| `azureCredentialId` | Azure credentials ID |
| `webappName` | Name of the Azure Web App |
| `resourceGroupName` | Azure resource group |
| `deploymentType` | `package` for ZIP deployment |
| `webappPackageType` | `zip` for ZIP packages |
| `webappCleanTargetPath` | Clean target before deploy |
| `webappTargetPath` | Target path on web app |
| `rollbackEnabled` | Enable rollback capability |
| `appSettingsEnabled` | Enable app settings override |
| `appSettings` | Array of app settings |
| `connectionStringEnabled` | Enable connection string override |
| `connectionStrings` | Array of connection strings |

---

## App Settings Configuration

### Override App Settings During Deployment

```json
{
  "appSettingsEnabled": true,
  "appSettings": [
    {
      "name": "NODE_ENV",
      "value": "production",
      "slotSetting": false
    },
    {
      "name": "API_URL",
      "value": "https://api.example.com",
      "slotSetting": false
    },
    {
      "name": "DATABASE_URL",
      "value": "@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/db-url/)",
      "slotSetting": false
    }
  ]
}
```

### Connection Strings

```json
{
  "connectionStringEnabled": true,
  "connectionStrings": [
    {
      "name": "DefaultConnection",
      "value": "[CONNECTION_STRING]",
      "type": "SQLAzure",
      "slotSetting": false
    }
  ]
}
```

---

## Extended Pipeline with Pre/Post Steps

### Full Pipeline with Validation and Testing

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Azure Web App deployment with validation and testing",
  "type": ["sdlc"],
  "tags": ["azure", "webapp", "nodejs"],
  "workflow": [
    {
      "_id": "677001a1b2c3d4e5f6a7b8c0",
      "name": "Pre-Build Validation",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "PreBuildValidation",
          "script": "#!/bin/bash\necho \"Validating project structure...\"\nif [ -f \"package.json\" ]; then\n  echo \"package.json found\"\n  cat package.json | grep '\"name\"'\nelse\n  echo \"ERROR: package.json not found\"\n  exit 1\nfi\necho \"Validation complete\"",
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
      "_id": "677001a1b2c3d4e5f6a7b8c1",
      "name": "Node.js Build",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c0"],
      "tool": {
        "tool_identifier": "command-line",
        "job_type": "shell script",
        "configuration": {
          "jobType": "SHELL SCRIPT",
          "buildType": "others",
          "commands": "apt update\napt install zip unzip\nnpm ci\nnpm run build\nzip -r nodeArtifact.zip .\nls -lrt",
          "outputFileName": "nodeArtifact.zip",
          "outputPath": ""
        }
      },
      "type": ["script"]
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c2",
      "name": "Azure Storage Push",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "azure-zip-deployment",
        "configuration": {
          "buildStepId": "677001a1b2c3d4e5f6a7b8c1",
          "containerName": "webapp-storage-package",
          "artifactName": "nodeArtifact.zip",
          "useRunCount": true
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c3",
      "name": "Azure Web App Deploy",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "azure_webapps",
        "configuration": {
          "artifactStepId": "677001a1b2c3d4e5f6a7b8c2",
          "deploymentType": "package",
          "webappPackageType": "zip",
          "webappCleanTargetPath": true
        }
      },
      "type": ["deploy"]
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c4",
      "name": "Post-Deploy Health Check",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c3"],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "HealthCheck",
          "script": "#!/bin/bash\necho \"Verifying Web App deployment...\"\nWEBAPP_URL=\"https://[WEBAPP_NAME].azurewebsites.net\"\necho \"Checking endpoint: $WEBAPP_URL\"\nsleep 30\nHTTP_CODE=$(curl -s -o /dev/null -w \"%{http_code}\" $WEBAPP_URL)\nif [ \"$HTTP_CODE\" -eq 200 ]; then\n  echo \"Web App is responding (HTTP $HTTP_CODE)\"\nelse\n  echo \"WARNING: Web App returned HTTP $HTTP_CODE\"\nfi\necho \"Health check complete\"",
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

## Multi-Environment Deployment

### Dev and Prod Pipeline

```json
{
  "workflow": [
    {
      "_id": "677001a1b2c3d4e5f6a7b8c1",
      "name": "Build",
      "active": true,
      "dependencies": [],
      "tool": {"tool_identifier": "command-line"}
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c2",
      "name": "Push to Storage",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c1"],
      "tool": {"tool_identifier": "azure-zip-deployment"}
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c3",
      "name": "Deploy to Dev",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "azure_webapps",
        "configuration": {
          "webappName": "[WEBAPP_NAME]-dev",
          "resourceGroupName": "[RESOURCE_GROUP]-dev"
        }
      }
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c4",
      "name": "Approval for Prod",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c3"],
      "tool": {
        "tool_identifier": "approval",
        "configuration": {
          "contact": "[APPROVER_EMAIL]",
          "message": "Approve deployment to Production?",
          "sendCustomMessage": true
        }
      }
    },
    {
      "_id": "677001a1b2c3d4e5f6a7b8c5",
      "name": "Deploy to Prod",
      "active": true,
      "dependencies": ["677001a1b2c3d4e5f6a7b8c4"],
      "tool": {
        "tool_identifier": "azure_webapps",
        "configuration": {
          "webappName": "[WEBAPP_NAME]-prod",
          "resourceGroupName": "[RESOURCE_GROUP]-prod"
        }
      }
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
    "pipelineId": "62e98c6a7a0577002de1be1a",
    "pipeline": {
      "_id": "62e98c6a7a0577002de1be1a",
      "name": "GOLDEN-TEMPLATE-NON-CONTAINER-APP-Node",
      "active": true,
      "type": ["sdlc"],
      "workflow": {
        "run_count": 880,
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
Pipeline ID: 62e98c6a7a0577002de1be1a
Portal URL: https://portal.opsera.io/workflow/details/62e98c6a7a0577002de1be1a/summary
```

Always include this URL in the response after successful pipeline creation.

---

## Implementation Instructions

### 1. Gather Requirements
- Pipeline name
- Source repository (GitHub/GitLab/Bitbucket)
- Runtime (Node.js, Java, Python, .NET)
- Azure Web App name
- Azure resource group
- Azure storage account and container
- Build dependencies and versions
- App settings to configure

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

1. **Use package deployment** for faster deployments
2. **Enable useRunCount** for artifact versioning in storage
3. **Clean target path** before deployment to avoid stale files
4. **Validate project structure** before building
5. **Add post-deployment health checks** to verify web app
6. **Use separate storage containers** for different environments
7. **Configure app settings** via pipeline for consistency
8. **Enable rollback** for production deployments

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `677001a1b2c3d4e5f6a7b8c1`

2. **"Missing buildStepId reference"**
   - Cause: Storage push step can't find build step
   - Fix: Ensure `buildStepId` matches the build step's `_id`

3. **"Missing artifactStepId reference"**
   - Cause: Deploy step can't find storage step
   - Fix: Ensure `artifactStepId` matches the storage step's `_id`

### Runtime Errors

1. **Azure authentication failure**: Verify Azure credentials are valid
2. **Storage container not found**: Check `existingContainer` setting or create container
3. **Web app not found**: Verify web app name and resource group
4. **Package deployment failed**: Check ZIP format and web.config settings
5. **Build dependencies missing**: Verify `dependencyType` configuration
6. **npm install fails**: Check package.json and node version compatibility

---

## Reference Pipeline

See pipeline `62e98c6a7a0577002de1be1a` (GOLDEN-TEMPLATE-NON-CONTAINER-APP-Node_DO NOT DEL) for a complete working example with 880 successful runs.

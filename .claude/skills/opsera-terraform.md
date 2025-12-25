# Opsera Terraform Pipeline Skill

> **Branding**: Follow the Opsera branding guidelines in `.claude/CLAUDE.md` for all responses.

## Description
Creates Opsera pipelines for Terraform infrastructure-as-code deployments across AWS, Azure, and GCP cloud providers.

## Usage
```
/opsera-terraform [cloud-provider] [options]
```

## Parameters
- `cloud-provider`: aws | azure | gcp (default: azure)
- `--name`: Pipeline name
- `--repo`: GitHub repository URL
- `--path`: Terraform files path in repository
- `--state`: remote | local (default: remote)
- `--notifications`: slack | teams | email (comma-separated)

## Examples
```
/opsera-terraform azure --name "My Azure Infra" --repo "org/iac-repo" --path "terraform/azure"
/opsera-terraform aws --name "AWS EKS Cluster" --state remote --notifications slack,teams
```

---

## Pipeline Template

When creating an Opsera Terraform pipeline, use the following structure:

### Step 1: Terraform Execution

```json
{
  "name": "Terraform [CLOUD_PROVIDER]",
  "active": true,
  "tool": {
    "tool_identifier": "terraform",
    "configuration": {
      "cloudProvider": "[aws|azure|gcp]",
      "backendState": "[AZURERM|S3|GCS|LOCAL]",
      "stateRemote": true,
      "toolActionType": "EXECUTE",
      "gitRepository": "[REPO_NAME]",
      "gitRepositoryID": "[ORG/REPO]",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitFilePath": "[TERRAFORM_PATH]",
      "defaultBranch": "main",
      "saveParameters": true,
      "customParameters": []
    }
  },
  "type": ["infrastructure-code"],
  "notification": []
}
```

### Cloud-Specific Backend Configuration

#### Azure (AZURERM)
```json
{
  "backendState": "AZURERM",
  "azureCredentialId": "[AZURE_CRED_ID]",
  "azureToolConfigId": "[AZURE_TOOL_ID]",
  "azureCPCredentialId": "[AZURE_CP_CRED_ID]",
  "azureCPToolConfigId": "[AZURE_CP_TOOL_ID]",
  "resourceGroup": "[RESOURCE_GROUP_NAME]",
  "storageName": "[STORAGE_ACCOUNT_NAME]",
  "containerName": "[CONTAINER_NAME]",
  "stateFile": "manual"
}
```

#### AWS (S3)
```json
{
  "backendState": "S3",
  "awsToolConfigId": "[AWS_TOOL_ID]",
  "awsCPToolConfigId": "[AWS_CP_TOOL_ID]",
  "bucketName": "[S3_BUCKET_NAME]",
  "bucketRegion": "[AWS_REGION]",
  "iamRoleFlag": false,
  "crossAccountIamRoleFlag": false
}
```

#### GCP (GCS)
```json
{
  "backendState": "GCS",
  "gcpToolConfigId": "[GCP_TOOL_ID]",
  "gcpCPToolConfigId": "[GCP_CP_TOOL_ID]",
  "bucketName": "[GCS_BUCKET_NAME]"
}
```

### Step 2: Post-Terraform CLI (Optional)

```json
{
  "name": "Post-Terraform CLI",
  "active": true,
  "dependencies": ["[TERRAFORM_STEP_ID]"],
  "tool": {
    "tool_identifier": "command-line",
    "configuration": {
      "jobType": "SHELL SCRIPT",
      "agentLabels": "generic-linux",
      "autoScaleEnable": true,
      "useTerraformOutput": true,
      "terraformStepId": "[TERRAFORM_STEP_ID]",
      "commands": "[SHELL_COMMANDS]",
      "customParameters": []
    }
  },
  "type": ["script"]
}
```

---

## Implementation Instructions

When the user requests a Terraform pipeline, follow these steps:

### 1. Gather Requirements
Ask for or determine:
- Cloud provider (AWS, Azure, GCP)
- Pipeline name
- Git repository containing Terraform code
- Path to Terraform files in repository
- State management preference (remote/local)
- Notification channels

### 2. Create Pipeline via API

Use the Opsera MCP tool to create the pipeline:

```
mcp__opsera-ai-agent__post_api_v2_pipeline_create
```

### 3. Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

| Format | Example | Valid? |
|--------|---------|--------|
| 24-char hex | `6767a1b2c3d4e5f6a7b8c9d0` | Yes |
| Descriptive | `terraform001aws000001` | No - API rejects |
| Short hex | `abc123` | No - must be 24 chars |

Generate unique IDs using pattern: `[6-char timestamp][18-char random hex]`
Example: `676701` + `a1b2c3d4e5f6a7b8c9` = `676701a1b2c3d4e5f6a7b8c9`

### 4. Required Parameters

#### For Azure Terraform Pipeline:
```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Terraform Azure infrastructure pipeline",
  "type": ["infrastructure-code"],
  "tags": ["terraform", "azure", "iac"],
  "workflow": [
    {
      "_id": "[GENERATE_24_CHAR_HEX]",
      "name": "Terraform Azure",
      "active": true,
      "tool": {
        "tool_identifier": "terraform",
        "configuration": {
          "cloudProvider": "azure",
          "backendState": "AZURERM",
          "stateRemote": true,
          "toolActionType": "EXECUTE",
          "gitFilePath": "[TERRAFORM_PATH]",
          "defaultBranch": "main",
          "saveParameters": true
        }
      },
      "type": ["infrastructure-code"]
    }
  ]
}
```

### 4. Output Parameters Configuration

To capture Terraform outputs and save them as Opsera parameters:

```json
{
  "saveParameters": true,
  "customParameters": [
    {
      "outputKey": "[TERRAFORM_OUTPUT_NAME]",
      "parameterId": "[OPSERA_PARAMETER_ID]",
      "parameterName": "[PARAMETER_NAME]"
    }
  ]
}
```

### 5. Notification Configuration

```json
{
  "notification": [
    {
      "type": "slack",
      "enabled": true,
      "event": "all",
      "channel": "[CHANNEL_NAME]",
      "toolId": "[SLACK_TOOL_ID]"
    },
    {
      "type": "teams",
      "enabled": true,
      "event": "all",
      "teamsType": "webhook",
      "toolId": "[TEAMS_TOOL_ID]"
    },
    {
      "type": "email",
      "enabled": true,
      "event": "error",
      "addresses": ["[EMAIL_LIST]"]
    }
  ]
}
```

---

## Complete Pipeline Examples

### Azure Terraform with Remote State

```json
{
  "name": "Terraform-Azure-Production",
  "description": "Production Azure infrastructure with remote state",
  "type": ["infrastructure-code"],
  "tags": ["terraform", "azure", "production"],
  "workflow": [
    {
      "_id": "6767a1b2c3d4e5f6a7b8c901",
      "name": "Terraform Azure",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "terraform",
        "configuration": {
          "cloudProvider": "azure",
          "backendState": "AZURERM",
          "stateRemote": true,
          "toolActionType": "EXECUTE",
          "gitRepository": "iac-examples",
          "gitRepositoryID": "myorg/iac-examples",
          "gitUrl": "https://github.com/myorg/iac-examples",
          "gitFilePath": "terraform/azure",
          "defaultBranch": "main",
          "resourceGroup": "terraform-state-rg",
          "storageName": "tfstatestorage",
          "containerName": "tfstate",
          "stateFile": "manual",
          "saveParameters": true,
          "customParameters": [
            {
              "outputKey": "resource_group_name",
              "parameterName": "opsera-azure-rg-name"
            },
            {
              "outputKey": "vm_public_ip",
              "parameterName": "opsera-vm-public-ip"
            }
          ]
        }
      },
      "type": ["infrastructure-code"],
      "notification": [
        {
          "type": "slack",
          "enabled": true,
          "event": "all",
          "channel": "infrastructure-alerts"
        }
      ]
    },
    {
      "_id": "6767a1b2c3d4e5f6a7b8c902",
      "name": "Post-Provisioning",
      "active": true,
      "dependencies": ["6767a1b2c3d4e5f6a7b8c901"],
      "tool": {
        "tool_identifier": "command-line",
        "configuration": {
          "jobType": "SHELL SCRIPT",
          "agentLabels": "generic-linux",
          "autoScaleEnable": true,
          "useTerraformOutput": true,
          "terraformStepId": "6767a1b2c3d4e5f6a7b8c901",
          "commands": "echo 'Infrastructure provisioned successfully'\necho 'Resource Group: ${opsera-azure-rg-name}'\necho 'VM IP: ${opsera-vm-public-ip}'"
        }
      },
      "type": ["script"]
    }
  ]
}
```

### AWS Terraform with S3 Backend

```json
{
  "name": "Terraform-AWS-EKS",
  "description": "AWS EKS cluster with S3 state backend",
  "type": ["infrastructure-code"],
  "tags": ["terraform", "aws", "eks"],
  "workflow": [
    {
      "_id": "6767a1b2c3d4e5f6a7b8c9d0",
      "name": "Terraform AWS",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "terraform",
        "configuration": {
          "cloudProvider": "aws",
          "backendState": "S3",
          "stateRemote": true,
          "toolActionType": "EXECUTE",
          "gitRepository": "aws-infrastructure",
          "gitRepositoryID": "myorg/aws-infrastructure",
          "gitUrl": "https://github.com/myorg/aws-infrastructure",
          "gitFilePath": "terraform/eks",
          "defaultBranch": "main",
          "bucketName": "terraform-state-bucket",
          "bucketRegion": "us-east-1",
          "saveParameters": true,
          "customParameters": []
        }
      },
      "type": ["infrastructure-code"]
    }
  ]
}
```

---

## API Response Structure

On successful pipeline creation, the API returns:

```json
{
  "status": "success",
  "data": {
    "pipelineId": "694a3a922c0fedbfcf510e13",
    "pipeline": {
      "_id": "694a3a922c0fedbfcf510e13",
      "name": "speri-infra",
      "description": "AWS Terraform infrastructure pipeline with S3 state backend",
      "active": true,
      "type": ["infrastructure-code"],
      "tags": ["terraform", "aws", "iac"],
      "owner": "68efe391f95f3e491f6d9002",
      "account": "org109-acc",
      "workflow": {
        "run_count": 0,
        "plan": [...]
      },
      "roles": [...],
      "createdAt": "2025-12-23T06:45:38.916Z"
    }
  },
  "metadata": {
    "message": "Pipeline created successfully",
    "apiVersion": "v2"
  }
}
```

**Key response fields:**
- `data.pipelineId` - Use this to reference the pipeline in subsequent operations
- `data.pipeline.workflow.plan` - Contains the created workflow steps
- `data.pipeline.roles` - Shows access permissions

---

## Opsera Portal URL

After creating a pipeline, provide the user with the direct link to view it in the Opsera portal:

```
https://portal.opsera.io/workflow/details/{pipelineId}/summary
```

**Example:**
```
Pipeline ID: 694a3a922c0fedbfcf510e13
Portal URL: https://portal.opsera.io/workflow/details/694a3a922c0fedbfcf510e13/summary
```

Always include this URL in the response after successful pipeline creation.

---

## Terraform Actions

The `toolActionType` supports the following operations:

| Action | Description |
|--------|-------------|
| EXECUTE | Run terraform init, plan, and apply |
| PLAN | Run terraform init and plan only |
| DESTROY | Run terraform destroy |

---

## Best Practices

1. **Always use remote state** for team collaboration
2. **Enable saveParameters** to capture Terraform outputs
3. **Configure notifications** for visibility
4. **Use specific terraform paths** rather than repository root
5. **Set up proper IAM/Service Principal** credentials before pipeline creation
6. **Use tags** for organization and filtering

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `6767a1b2c3d4e5f6a7b8c9d0`

2. **"Pipeline plan structure validation failed"**
   - Cause: Invalid workflow structure or missing required fields
   - Fix: Ensure each step has `_id`, `name`, `active`, and `tool` fields

3. **"Invalid tool_identifier"**
   - Cause: Unsupported tool type specified
   - Fix: Use `terraform` for Terraform steps, `command-line` for CLI steps

### Common Runtime Issues

1. **State Lock Error**: Check if another process is running or manually release the lock
2. **Credential Error**: Verify cloud provider credentials are properly configured in Opsera
3. **Backend Init Failure**: Ensure storage account/bucket exists and is accessible
4. **Git Clone Failure**: Verify repository access and credentials

### Debug Commands
```bash
# Check terraform version
terraform version

# Validate configuration
terraform validate

# Format check
terraform fmt -check
```

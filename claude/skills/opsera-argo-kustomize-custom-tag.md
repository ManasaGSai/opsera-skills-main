# Opsera ArgoCD Kustomize Pipeline with Custom Tagging

## Skill Overview
Create ArgoCD deployment pipelines with Kustomize support, custom image tagging, and AWS ECR integration. Supports optional approval gates and blue-green deployments.

## Use Cases
- GitOps-based continuous deployment with ArgoCD
- Kustomize-based Kubernetes configuration management
- Custom Docker image tag workflows
- AWS ECR container registry integration
- Manual approval gates for production deployments
- Blue-green deployment strategies

## Pipeline Pattern
Based on production pipeline: `REG_SCH_Argo_AWS_CustomTag` (ID: 6489bea6a16c100012860100)

## Required Information

### 1. Git Repository Configuration
- **Repository URL**: GitHub/GitLab/Bitbucket repository with Kustomize configs
- **Branch**: Target branch (e.g., `kustomize_example`)
- **Kustomize Path**: Path to kustomization.yaml file (e.g., `yaml/dev/kustomization.yaml`)
- **Git Tool ID**: Opsera tool configuration for Git access

### 2. AWS ECR Configuration
- **ECR Repository Name**: Target ECR repository
- **AWS Tool Config ID**: Opsera AWS credential configuration
- **Custom Image Tag**: Enable custom tagging (recommended)
- **Image Reference**: Name reference in Kustomize files

### 3. ArgoCD Configuration
- **Tool Config ID**: Opsera ArgoCD tool configuration
- **Application Name**: ArgoCD application name
- **Tool URL**: ArgoCD server URL
- **Delay Time**: Sync delay in seconds (default: 10)

### 4. Optional Components
- **Approval Gate**: Manual approval before deployment
- **Slack Notifications**: Deployment status notifications
- **Blue-Green Deployment**: Optional second stage for canary/blue-green
- **Rollback**: Enable automatic rollback on failure

## Pipeline Structure

```
┌─────────────────────────────────────────────────────────┐
│  STEP 1: Approval Gate (Optional)                       │
├─────────────────────────────────────────────────────────┤
│  • Manual approval required                             │
│  • Slack notification to team                           │
│  • Custom approval message                              │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  STEP 2: ArgoCD Kustomize Deployment                    │
├─────────────────────────────────────────────────────────┤
│  • Clone Git repository                                 │
│  • Update Kustomize image reference                     │
│  • Custom tag from parameter/ECR                        │
│  • Trigger ArgoCD sync                                  │
│  • Wait for deployment completion                       │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  STEP 3: Blue-Green Deployment (Optional)               │
├─────────────────────────────────────────────────────────┤
│  • Deploy to preview environment                        │
│  • Manual promotion to production                       │
│  • Rollback capability                                  │
└─────────────────────────────────────────────────────────┘
```

## Example Request

**User**: "Create an ArgoCD pipeline with Kustomize for my Node.js app using custom tags from ECR"

**Assistant Response**:
```
⚡ OPSERA │ ArgoCD Kustomize Custom Tag Pipeline
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

I'll create an ArgoCD pipeline with Kustomize and custom tagging.

Please provide:

1. Git Repository:
   - Repository URL (e.g., github.com/org/repo)
   - Branch name
   - Path to kustomization.yaml

2. AWS ECR:
   - ECR repository name
   - Image tag format (or use dynamic tagging)

3. ArgoCD:
   - Application name in ArgoCD
   - Cluster name (if multiple)

4. Optional:
   - Enable approval gate? (yes/no)
   - Enable Slack notifications? (yes/no)
   - Enable blue-green deployment? (yes/no)
```

## Pipeline Configuration Template

```json
{
  "name": "[APP_NAME] ArgoCD Kustomize Deployment",
  "description": "GitOps deployment with Kustomize and custom image tagging",
  "active": true,
  "type": ["sfdc"],
  "tags": [
    {"type": "pipeline", "value": "argocd"},
    {"type": "pipeline", "value": "kustomize"},
    {"type": "pipeline", "value": "custom-tag"}
  ],
  "workflow": [
    {
      "name": "Approval Gate",
      "active": false,
      "dependencies": [],
      "tool": {
        "tool_identifier": "approval",
        "configuration": {
          "contact": "devops_team",
          "message": "Approve deployment to [ENVIRONMENT]",
          "sendCustomMessage": true
        },
        "threshold": {
          "type": "user-approval",
          "value": {
            "approved": false,
            "denied": false
          }
        }
      },
      "type": ["interaction-gate"],
      "notification": [
        {
          "type": "slack",
          "enabled": true,
          "event": "all",
          "logEnabled": true,
          "channel": "deployments",
          "toolId": "[SLACK_TOOL_ID]"
        }
      ]
    },
    {
      "name": "ArgoCD Kustomize Deploy",
      "active": true,
      "dependencies": ["Approval Gate"],
      "tool": {
        "tool_identifier": "argo",
        "configuration": {
          "type": "github",
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[GIT_URL]",
          "gitRepository": "[REPO_NAME]",
          "gitRepositoryID": "[ORG/REPO]",
          "defaultBranch": "[BRANCH]",
          "gitFilePath": "[KUSTOMIZE_PATH]",
          "sshUrl": "[GIT_SSH_URL]",

          "kustomizeFlag": true,
          "helmFlag": false,
          "customImageTag": true,
          "useParameterForRepositoryTag": true,

          "platform": "aws",
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "ecrRepoName": "[ECR_REPO_NAME]",
          "imageReference": "[IMAGE_NAME]",
          "existingContent": "image",

          "toolConfigId": "[ARGO_TOOL_ID]",
          "toolUrl": "[ARGO_URL]",
          "applicationName": "[ARGO_APP_NAME]",
          "userName": "admin",
          "delayTime": "10",

          "dynamicVariables": false,
          "rollbackEnabled": false,
          "isBlueGreenDeployment": false
        }
      },
      "type": ["deploy"]
    }
  ]
}
```

## Key Features Explained

### 1. Custom Image Tagging
```json
"customImageTag": true,
"useParameterForRepositoryTag": true,
"repositoryTag": "[TAG_PARAMETER]"
```
- Allows dynamic image tags from parameters
- Can use build numbers, commit SHAs, or custom versioning
- Updates Kustomize image reference automatically

### 2. Kustomize Integration
```json
"kustomizeFlag": true,
"gitFilePath": "yaml/dev/kustomization.yaml",
"existingContent": "image",
"imageReference": "my-app"
```
- Modifies kustomization.yaml image section
- Supports overlays (dev/staging/prod)
- No manual YAML editing required

### 3. AWS ECR Integration
```json
"platform": "aws",
"awsToolConfigId": "[ID]",
"ecrRepoName": "my-ecr-repo"
```
- Authenticates with AWS
- Pulls from private ECR repositories
- Supports cross-region deployments

### 4. ArgoCD Sync
```json
"toolConfigId": "[ARGO_TOOL_ID]",
"applicationName": "my-app",
"delayTime": "10"
```
- Triggers ArgoCD application sync
- Waits for sync completion
- Configurable delay for health checks

## Optional: Blue-Green Deployment

Add a second ArgoCD step for blue-green:

```json
{
  "name": "Blue-Green Deployment",
  "active": true,
  "dependencies": ["ArgoCD Kustomize Deploy"],
  "tool": {
    "tool_identifier": "argo",
    "configuration": {
      "isBlueGreenDeployment": true,
      "applicationName": "[APP_NAME]-bluegreen",
      "gitFilePath": "blue-green/rollout.yaml",
      "rollbackEnabled": true
    }
  },
  "type": ["deploy"]
}
```

## Tool Identifiers Required

| Tool | Purpose | How to Get |
|------|---------|------------|
| Git Tool ID | Repository access | List tools: `/api/v2/tools?type=git` |
| AWS Tool ID | ECR authentication | List tools: `/api/v2/tools?type=aws` |
| ArgoCD Tool ID | ArgoCD server | List tools: `/api/v2/tools?type=argo` |
| Slack Tool ID (optional) | Notifications | List tools: `/api/v2/tools?type=slack` |

## Best Practices

### 1. Custom Tag Naming
Use semantic versioning or commit-based tags:
- `v1.2.3` - Semantic version
- `main-abc123` - Branch + commit SHA
- `20240615-76` - Date + build number

### 2. Kustomize Structure
Organize by environment:
```
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
```

### 3. Approval Gates
Enable approval for:
- Production deployments
- Infrastructure changes
- Major version updates

### 4. Notifications
Configure Slack for:
- Deployment start
- Deployment success
- Deployment failures
- Approval requests

## Example Kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml

images:
  - name: the-my-app
    newName: 472496548172.dkr.ecr.us-east-2.amazonaws.com/aws-helm-multiplefile
    newTag: custom-tag-here
```

Opsera automatically updates the `newTag` field with your custom tag.

## Troubleshooting

### Issue: ArgoCD sync fails
**Solution**: Check ArgoCD application exists and credentials are valid

### Issue: Image not found in ECR
**Solution**: Verify ECR repository name and AWS credentials

### Issue: Kustomize path not found
**Solution**: Check gitFilePath matches your repository structure

### Issue: Custom tag not applied
**Solution**: Ensure `customImageTag: true` and `useParameterForRepositoryTag: true`

## Related Skills
- `/opsera-argo-container` - Standard ArgoCD deployments
- `/opsera-multicloud-container` - Multi-cloud with security
- `/opsera-terraform` - Infrastructure provisioning before deployment

## Production Stats
- **Pipeline ID**: 6489bea6a16c100012860100
- **Run Count**: 486+ successful deployments
- **Last Run**: Success
- **Trigger Type**: Scheduled
- **Organization**: Production QA environment

---

**Skill Created By**: Opsera AI Assistant
**Based On**: Production pipeline `REG_SCH_Argo_AWS_CustomTag`
**Last Updated**: 2025-12-23
**Version**: 1.0

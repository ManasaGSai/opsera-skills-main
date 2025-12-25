# Opsera Lineaje Security Scanning with Automated Patching and Deployment

## Skill Overview
Create security-first container pipelines with Lineaje vulnerability scanning, automated image patching using Copacetic, re-scanning for verification, and secure deployment with ArgoCD. This pipeline ensures only patched, secure images reach production.

## Use Cases
- Security-first container deployment workflows
- Automated vulnerability remediation with Copacetic
- Container image scanning and validation with Lineaje
- Shift-left security in CI/CD pipelines
- Compliance-driven deployments requiring verified security posture
- Zero-vulnerability deployment strategies

## Pipeline Pattern
Based on production pipeline: `REG_SCH_Lineaje` (ID: 6793ee0d8fc4ae00126fe980)

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  🔧 BUILD: Docker Build & Push to ECR                   │
├─────────────────────────────────────────────────────────┤
│  • Clone source repository                              │
│  • Build Docker image                                   │
│  • Tag with timestamp/commit                            │
│  • Push to AWS ECR                                      │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  🔍 SCAN: Lineaje Security Scan (Original Image)        │
├─────────────────────────────────────────────────────────┤
│  • Pull image from ECR                                  │
│  • Deep vulnerability analysis                          │
│  • Policy compliance check                              │
│  • Generate remediation report                          │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  🔧 PATCH: Copacetic Automated Patching                 │
├─────────────────────────────────────────────────────────┤
│  • Analyze Lineaje scan results                         │
│  • Auto-patch OS-level vulnerabilities                  │
│  • Create patched image variant                         │
│  • Push patched image to ECR                            │
│  • Tag: {original-tag}-patched                          │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  ✅ VERIFY: Lineaje Re-Scan (Patched Image)             │
├─────────────────────────────────────────────────────────┤
│  • Scan patched image                                   │
│  • Verify vulnerability reduction                       │
│  • Validate policy compliance                           │
│  • Confirm security posture                             │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  🚀 DEPLOY: ArgoCD GitOps Deployment                    │
├─────────────────────────────────────────────────────────┤
│  • Use patched image from Lineaje step                  │
│  • Update Kubernetes manifests                          │
│  • Trigger ArgoCD sync                                  │
│  • Deploy verified secure image                         │
└─────────────────────────────────────────────────────────┘
```

## Required Information

### 1. Source Repository Configuration
- **Repository URL**: GitHub/GitLab/Bitbucket repository with Dockerfile
- **Branch**: Target branch (e.g., `main`, `develop`)
- **Git Tool ID**: Opsera tool configuration for Git access
- **Dockerfile Location**: Path to Dockerfile (default: root)

### 2. AWS ECR Configuration
- **ECR Repository Name**: Target ECR repository for images
- **AWS Tool Config ID**: Opsera AWS credential configuration
- **AWS Region**: ECR region (e.g., `us-east-2`)
- **Tag Strategy**: Timestamp, commit SHA, or semantic versioning

### 3. Lineaje Security Configuration
- **Tool Config ID**: Opsera Lineaje tool configuration
- **Organization ID**: Lineaje organization identifier
- **Organization Name**: Lineaje organization name
- **Project Name**: Lineaje project for tracking scans
- **Policy Name (Original)**: Policy for initial scan (e.g., "Opsera Policy 1")
- **Policy Name (Patched)**: Policy for patched image scan (e.g., "Opsera Policy 2")
- **Scan Type**: `container_registries` for ECR scanning

### 4. Copacetic Patching Configuration
- **Lineaje Step ID**: Reference to Lineaje scan step
- **Patching Strategy**: Automatic OS-level vulnerability patching
- **Output Tag Suffix**: Default: `-patched`

### 5. ArgoCD Deployment Configuration
- **Tool Config ID**: Opsera ArgoCD tool configuration
- **Application Name**: ArgoCD application name
- **Git Repository**: Kubernetes manifest repository
- **Manifest Path**: Path to deployment YAML (e.g., `yaml/deployment.yaml`)
- **Deployment Strategy**: Direct image update

## Pipeline Structure

### Step 1: Docker Build & Push
```json
{
  "name": "Docker Build & Push",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "docker-cli",
    "configuration": {
      "service": "github",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitBranch": "main",
      "repoId": "[ORG/REPO]",
      "enableDockerBuild": true,
      "enableDockerPush": true,
      "registryType": "ecr",
      "awsToolConfigId": "[AWS_TOOL_ID]",
      "dockerRegistryName": "[AWS_ACCOUNT].dkr.ecr.[REGION].amazonaws.com/[REPO_NAME]",
      "repositoryName": "[ECR_REPO_NAME]",
      "dockerTagType": "timestamp",
      "dynamicTag": true,
      "useDockerDynamicName": true,
      "dockerDynamicName": "branch_name"
    }
  },
  "type": ["build"]
}
```

### Step 2: Lineaje Security Scan (Original)
```json
{
  "name": "Lineaje Security Scan",
  "active": true,
  "dependencies": ["Docker Build & Push"],
  "tool": {
    "tool_identifier": "lineaje",
    "configuration": {
      "toolConfigId": "[LINEAJE_TOOL_ID]",
      "organizationId": "[LINEAJE_ORG_ID]",
      "organizationName": "[LINEAJE_ORG_NAME]",
      "projectName": "[LINEAJE_PROJECT]",
      "policyName": "Opsera Policy 1",
      "scanType": "container_registries",
      "artifactStepId": "[DOCKER_STEP_ID]",
      "patchedImage": false,
      "customImageTag": false
    }
  },
  "type": ["container-scan"]
}
```

### Step 3: Copacetic Image Patching
```json
{
  "name": "Copacetic Image Patch",
  "active": true,
  "dependencies": ["Lineaje Security Scan"],
  "tool": {
    "tool_identifier": "copacetic",
    "configuration": {
      "lineajeStepId": "[LINEAJE_STEP_ID]"
    }
  },
  "type": ["patch_container_images"]
}
```

### Step 4: Lineaje Re-Scan (Patched)
```json
{
  "name": "Lineaje Patch Verification",
  "active": true,
  "dependencies": ["Copacetic Image Patch"],
  "tool": {
    "tool_identifier": "lineaje",
    "configuration": {
      "toolConfigId": "[LINEAJE_TOOL_ID]",
      "organizationId": "[LINEAJE_ORG_ID]",
      "organizationName": "[LINEAJE_ORG_NAME]",
      "projectName": "[LINEAJE_PROJECT]",
      "policyName": "Opsera Policy 2",
      "scanType": "container_registries",
      "artifactStepId": "[DOCKER_STEP_ID]",
      "patchedImage": true,
      "customImageTag": false
    }
  },
  "type": ["container-scan"]
}
```

### Step 5: ArgoCD Secure Deployment
```json
{
  "name": "ArgoCD Secure Deploy",
  "active": true,
  "dependencies": ["Lineaje Patch Verification"],
  "tool": {
    "tool_identifier": "argo",
    "configuration": {
      "toolConfigId": "[ARGO_TOOL_ID]",
      "toolUrl": "[ARGO_URL]",
      "applicationName": "[ARGO_APP_NAME]",
      "type": "github",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[MANIFEST_REPO_URL]",
      "gitRepository": "[MANIFEST_REPO]",
      "gitRepositoryID": "[ORG/MANIFEST_REPO]",
      "defaultBranch": "main",
      "gitFilePath": "yaml/deployment.yaml",
      "dockerStepID": "[LINEAJE_PATCH_STEP_ID]",
      "dockerStepType": "lineaje",
      "existingContent": "image",
      "delayTime": "5",
      "kustomizeFlag": false,
      "helmFlag": false
    }
  },
  "type": ["deploy"]
}
```

## Complete Pipeline Template

```json
{
  "name": "[APP_NAME] Lineaje Security Pipeline",
  "description": "Security-first pipeline with Lineaje scanning, Copacetic patching, and ArgoCD deployment",
  "active": true,
  "type": ["sfdc"],
  "tags": [
    {"type": "pipeline", "value": "lineaje"},
    {"type": "pipeline", "value": "copacetic"},
    {"type": "pipeline", "value": "security"},
    {"type": "pipeline", "value": "argocd"}
  ],
  "workflow": [
    {
      "name": "Docker Build & Push",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "docker-cli",
        "configuration": {
          "service": "github",
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[REPO_URL]",
          "gitBranch": "main",
          "repoId": "[ORG/REPO]",
          "gitRepository": "[REPO_NAME]",
          "sshUrl": "[SSH_URL]",
          "enableDockerBuild": true,
          "enableDockerPush": true,
          "registryType": "ecr",
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "dockerRegistryName": "[AWS_ACCOUNT].dkr.ecr.[REGION].amazonaws.com/[REPO_NAME]",
          "repositoryName": "[ECR_REPO_NAME]",
          "dockerTagType": "timestamp",
          "dynamicTag": true,
          "useDockerDynamicName": true,
          "dockerDynamicName": "branch_name",
          "dockerName": "main"
        }
      },
      "type": ["build"]
    },
    {
      "name": "Lineaje Security Scan",
      "active": true,
      "dependencies": ["Docker Build & Push"],
      "tool": {
        "tool_identifier": "lineaje",
        "configuration": {
          "toolConfigId": "[LINEAJE_TOOL_ID]",
          "organizationId": "[LINEAJE_ORG_ID]",
          "organizationName": "[LINEAJE_ORG_NAME]",
          "projectName": "[LINEAJE_PROJECT]",
          "policyName": "Opsera Policy 1",
          "scanType": "container_registries",
          "artifactStepId": "[DOCKER_STEP_ID]",
          "patchedImage": false,
          "customImageTag": false
        }
      },
      "type": ["container-scan"]
    },
    {
      "name": "Copacetic Image Patch",
      "active": true,
      "dependencies": ["Lineaje Security Scan"],
      "tool": {
        "tool_identifier": "copacetic",
        "configuration": {
          "lineajeStepId": "[LINEAJE_STEP_ID]"
        }
      },
      "type": ["patch_container_images"]
    },
    {
      "name": "Lineaje Patch Verification",
      "active": true,
      "dependencies": ["Copacetic Image Patch"],
      "tool": {
        "tool_identifier": "lineaje",
        "configuration": {
          "toolConfigId": "[LINEAJE_TOOL_ID]",
          "organizationId": "[LINEAJE_ORG_ID]",
          "organizationName": "[LINEAJE_ORG_NAME]",
          "projectName": "[LINEAJE_PROJECT]",
          "policyName": "Opsera Policy 2",
          "scanType": "container_registries",
          "artifactStepId": "[DOCKER_STEP_ID]",
          "patchedImage": true,
          "customImageTag": false
        }
      },
      "type": ["container-scan"]
    },
    {
      "name": "ArgoCD Secure Deploy",
      "active": true,
      "dependencies": ["Lineaje Patch Verification"],
      "tool": {
        "tool_identifier": "argo",
        "configuration": {
          "toolConfigId": "[ARGO_TOOL_ID]",
          "toolUrl": "[ARGO_URL]",
          "applicationName": "[ARGO_APP_NAME]",
          "userName": "admin",
          "type": "github",
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[MANIFEST_REPO_URL]",
          "gitRepository": "[MANIFEST_REPO]",
          "gitRepositoryID": "[ORG/MANIFEST_REPO]",
          "sshUrl": "[MANIFEST_SSH_URL]",
          "defaultBranch": "main",
          "gitFilePath": "yaml/deployment.yaml",
          "dockerStepID": "[LINEAJE_PATCH_STEP_ID]",
          "dockerStepType": "lineaje",
          "existingContent": "image",
          "delayTime": "5",
          "kustomizeFlag": false,
          "helmFlag": false,
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

### 1. Lineaje Security Scanning
**What it does**:
- Deep vulnerability analysis of container images
- Software Bill of Materials (SBOM) generation
- Policy-based compliance checking
- Vulnerability prioritization and risk scoring
- Remediation guidance

**Configuration**:
```json
{
  "organizationId": "vdna_xxxxx",
  "organizationName": "company-name",
  "projectName": "app-project",
  "policyName": "Opsera Policy 1",
  "scanType": "container_registries"
}
```

### 2. Copacetic Automated Patching
**What it does**:
- Analyzes Lineaje scan results
- Identifies patchable OS-level vulnerabilities
- Generates patched container layers
- Creates new image with `-patched` suffix
- Preserves application code and dependencies

**How it works**:
1. Reads vulnerability data from Lineaje scan
2. Downloads OS packages with security fixes
3. Rebuilds affected container layers
4. Pushes patched image to same registry
5. Tags with original tag + `-patched`

Example:
- Original: `440953937617.dkr.ecr.us-east-2.amazonaws.com/app:20251222203206`
- Patched: `440953937617.dkr.ecr.us-east-2.amazonaws.com/app:20251222203206-patched`

### 3. Two-Stage Lineaje Scanning
**Why scan twice?**
- **First Scan**: Baseline vulnerability assessment
- **Second Scan**: Verify patching effectiveness

**Configuration Differences**:
```json
// First Scan
{
  "policyName": "Opsera Policy 1",
  "patchedImage": false
}

// Second Scan
{
  "policyName": "Opsera Policy 2",
  "patchedImage": true
}
```

### 4. Secure ArgoCD Deployment
**What makes it secure**:
- Only deploys patched, verified images
- Uses `dockerStepType: "lineaje"` to reference patched image
- Automatic manifest updates
- GitOps-based deployment

## Example Request

**User**: "Create a secure pipeline with Lineaje scanning and automated patching for my Node.js API"

**Assistant Response**:
```
⚡ OPSERA │ Lineaje Security Pipeline with Auto-Patching
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

I'll create a security-first pipeline with:
✓ Docker build & push to ECR
✓ Lineaje vulnerability scanning
✓ Copacetic automated patching
✓ Lineaje re-scan for verification
✓ ArgoCD secure deployment

Please provide:

1. Source Repository:
   - Repository URL
   - Branch name
   - Path to Dockerfile

2. AWS ECR:
   - ECR repository name
   - AWS region

3. Lineaje:
   - Organization ID
   - Organization name
   - Project name

4. ArgoCD:
   - Application name
   - Kubernetes manifest repository
   - Path to deployment YAML
```

## Tool Identifiers Required

| Tool | Purpose | Configuration |
|------|---------|---------------|
| Git Tool ID | Source code access | `/api/v2/tools?type=git` |
| AWS Tool ID | ECR authentication | `/api/v2/tools?type=aws` |
| Lineaje Tool ID | Security scanning | `/api/v2/tools?type=lineaje` |
| ArgoCD Tool ID | GitOps deployment | `/api/v2/tools?type=argo` |

## Best Practices

### 1. Policy Configuration
Create separate Lineaje policies:
- **Policy 1**: Strict baseline (fails on high/critical)
- **Policy 2**: Relaxed for patched images (informational)

### 2. Tag Naming Strategy
Use timestamps for traceability:
- Format: `YYYYMMDDHHMMSS`
- Example: `20251222203206`
- Patched: `20251222203206-patched`

### 3. Deployment Strategy
Always deploy patched images:
```json
{
  "dockerStepID": "[LINEAJE_PATCH_STEP_ID]",
  "dockerStepType": "lineaje"
}
```

### 4. Multi-Environment
Create variants for each environment:
- `dev-lineaje-pipeline`
- `staging-lineaje-pipeline`
- `prod-lineaje-pipeline`

### 5. Notification Strategy
Configure notifications for:
- High/critical vulnerabilities found
- Patching failures
- Deployment to production

## Security Benefits

### Vulnerability Reduction
- **Before Patching**: 50-100+ OS vulnerabilities typical
- **After Patching**: 60-80% reduction in patchable vulnerabilities
- **Zero-Day Protection**: Rapid patching for known CVEs

### Compliance
- Automated security gates
- Policy-based approvals
- Full audit trail with Lineaje
- SBOM generation for each image

### Shift-Left Security
- Security integrated into CI/CD
- Automated remediation
- No manual patching required
- Continuous vulnerability management

## Troubleshooting

### Issue: Copacetic patching fails
**Cause**: Not all vulnerabilities are patchable
**Solution**: Copacetic only patches OS-level vulnerabilities. Application dependencies require code changes.

### Issue: Lineaje scan timeout
**Cause**: Large images or slow network
**Solution**: Increase step timeout in pipeline settings

### Issue: ArgoCD doesn't use patched image
**Cause**: Incorrect `dockerStepType` configuration
**Solution**: Ensure `dockerStepType: "lineaje"` and `dockerStepID` references the **second** Lineaje scan step

### Issue: Patched image not found
**Cause**: Copacetic step failed silently
**Solution**: Check Copacetic logs and verify Lineaje scan completed successfully

## Advanced Configuration

### Conditional Patching
Only patch if vulnerabilities exceed threshold:
```json
{
  "threshold": {
    "completeFirst": true,
    "ensureSuccess": true
  }
}
```

### Multiple Policy Checks
Use different policies for different severity levels:
- Policy 1: Block on critical/high
- Policy 2: Warn on medium
- Policy 3: Informational for low

### Integration with Jira
Auto-create tickets for unpatchable vulnerabilities:
```json
{
  "notification": [
    {
      "type": "jira",
      "enabled": true,
      "event": "error",
      "jiraProject": "SEC",
      "jiraPriority": "High"
    }
  ]
}
```

## Deployment Flow Diagram

```
┌──────────────────┐
│  Git Push Event  │
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Docker Build    │ ← Source Code
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Push to ECR     │ ← Original Image
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Lineaje Scan    │ ← Baseline Scan
└────────┬─────────┘
         │
         ↓
    ❌ Vulnerabilities Found ❌
         │
         ↓
┌──────────────────┐
│  Copacetic Patch │ ← Auto-Remediate
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Push Patched    │ ← Patched Image
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Lineaje Re-Scan │ ← Verify Patch
└────────┬─────────┘
         │
         ↓
    ✅ Verified Secure ✅
         │
         ↓
┌──────────────────┐
│  ArgoCD Deploy   │ ← Production
└──────────────────┘
```

## Related Skills
- `/opsera-argo-container` - Standard ArgoCD deployments
- `/opsera-multicloud-container` - Multi-cloud with alternative security tools
- `/opsera-argo-kustomize-custom-tag` - Kustomize-based deployments

## Production Stats
- **Pipeline ID**: 6793ee0d8fc4ae00126fe980
- **Pipeline Name**: REG_SCH_Lineaje
- **Run Count**: 356+ successful deployments
- **Last Run**: Success
- **Success Rate**: Production-proven pattern

## Lineaje + Copacetic Integration

This pipeline leverages the powerful combination of:
- **Lineaje**: Enterprise-grade vulnerability intelligence
- **Copacetic**: Microsoft's open-source patching tool
- **Opsera**: Unified orchestration and automation

Result: **Zero-touch vulnerability remediation** from code to production.

---

**Skill Created By**: Opsera AI Assistant
**Based On**: Production pipeline `REG_SCH_Lineaje`
**Last Updated**: 2025-12-23
**Version**: 1.0
**Category**: Security, Container Security, DevSecOps

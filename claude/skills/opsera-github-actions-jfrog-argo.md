# Opsera GitHub Actions Integration with JFrog and ArgoCD Deployment

## Skill Overview
Create hybrid CI/CD pipelines that leverage existing GitHub Actions workflows for build automation, while using Opsera for orchestration, approval gates, and ArgoCD deployments with JFrog Artifactory. Perfect for organizations wanting to maintain GitHub Actions CI while gaining centralized governance and GitOps deployment capabilities.

## Use Cases
- Leverage existing GitHub Actions workflows within Opsera pipelines
- Centralized orchestration across multiple CI/CD tools
- Approval gates for production deployments
- JFrog Artifactory integration for enterprise artifact management
- GitOps-based Kubernetes deployments with ArgoCD
- Hybrid CI/CD strategy (GitHub Actions CI + Opsera CD)
- Multi-repo coordination with unified visibility

## Pipeline Pattern
Based on production pipeline: `Cisco_GitHubActions_Argo_Nagalakshmi_DND` (ID: 69425543d08886acba59140e)

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  🔧 CI: GitHub Actions Workflow                         │
├─────────────────────────────────────────────────────────┤
│  • Runs natively in GitHub                              │
│  • Build, test, security scans                          │
│  • Push artifacts to JFrog Artifactory                  │
│  • Triggered and monitored by Opsera                    │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  ⏸️ GATE: Manual Approval                                │
├─────────────────────────────────────────────────────────┤
│  • Human approval required                              │
│  • Custom approval message                              │
│  • Production deployment gate                           │
│  • Compliance & governance checkpoint                   │
└─────────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────┐
│  🚀 CD: ArgoCD GitOps Deployment                        │
├─────────────────────────────────────────────────────────┤
│  • Pull artifact from JFrog Artifactory                 │
│  • Update Kubernetes manifests                          │
│  • Trigger ArgoCD sync                                  │
│  • Deploy to Kubernetes cluster                         │
└─────────────────────────────────────────────────────────┘
```

## Why This Pattern?

### ✅ Best of Both Worlds

| Aspect | GitHub Actions | Opsera |
|--------|---------------|--------|
| **Build/Test** | ✅ Native GitHub integration | ✅ Orchestration & visibility |
| **Flexibility** | ✅ Vast marketplace | ✅ Enterprise governance |
| **Cost** | ✅ Free tier available | ✅ Unified platform |
| **Approvals** | ❌ Limited | ✅ Built-in approval gates |
| **Multi-tool** | ❌ Single platform | ✅ Cross-platform orchestration |
| **Visibility** | ❌ Per-repo | ✅ Unified dashboard |

### 🎯 Key Benefits

1. **Preserve Existing Investments**: Keep GitHub Actions workflows already in use
2. **Centralized Governance**: Opsera provides approval gates and unified visibility
3. **Enterprise Artifact Management**: JFrog Artifactory for secure, compliant storage
4. **GitOps Deployment**: ArgoCD ensures declarative, auditable deployments
5. **Flexibility**: Mix and match tools based on team preferences

## Required Information

### 1. GitHub Actions Configuration
- **Repository**: GitHub repository with existing workflow
- **Workflow ID**: GitHub Actions workflow ID (numeric)
- **Branch**: Branch to trigger (e.g., `main`, `develop`)
- **Git Tool ID**: Opsera GitHub tool configuration
- **Workflow Inputs**: Optional parameters to pass to the workflow

### 2. JFrog Artifactory Configuration
- **Tool Config ID**: Opsera JFrog Artifactory tool configuration
- **Repository Name**: JFrog repository (e.g., `nlp-docker`)
- **Repository Type**: Usually `REPOPATHPREFIX`
- **Image Tag**: Full image path with tag (e.g., `trial0no4i8.jfrog.io/nlp-docker/app:latest`)

### 3. ArgoCD Configuration
- **Tool Config ID**: Opsera ArgoCD tool configuration
- **Application Name**: ArgoCD application name
- **Git Repository**: Kubernetes manifest repository
- **Manifest Path**: Path to deployment YAML (e.g., `k8s/deployment.yaml`)
- **Platform**: Set to `jfrog` when using JFrog Artifactory

### 4. Approval Configuration
- **Contact**: Person/team responsible for approval
- **Message**: Custom approval message
- **Approval Required**: Boolean flag

## Pipeline Structure

### Step 1: GitHub Actions Workflow Trigger

```json
{
  "name": "CI-GitHub Actions",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "github_actions_workflow",
    "configuration": {
      "toolId": "[GITHUB_TOOL_ID]",
      "repositoryId": "[ORG/REPO]",
      "workflowId": "216591844",
      "branchOrTagName": "main",
      "job": "",
      "inputs": {}
    }
  },
  "type": ["git_automation"]
}
```

**What happens**:
- Opsera triggers the GitHub Actions workflow
- Workflow runs in GitHub (build, test, push to JFrog)
- Opsera monitors workflow execution
- Waits for completion before proceeding

### Step 2: Approval Gate

```json
{
  "name": "Approval for Prod Deploy",
  "active": true,
  "dependencies": ["CI-GitHub Actions"],
  "tool": {
    "tool_identifier": "approval",
    "configuration": {
      "contact": "DevOps Team",
      "message": "Approve deployment to production",
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
  "type": ["interaction-gate"]
}
```

**What happens**:
- Pipeline pauses after GitHub Actions completes
- Notification sent to designated approver
- Deployment blocked until approval granted
- Audit trail of who approved and when

### Step 3: ArgoCD Deployment

```json
{
  "name": "ArgoCD Deploy",
  "active": true,
  "dependencies": ["Approval for Prod Deploy"],
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
      "defaultBranch": "main",
      "gitFilePath": "k8s/deployment.yaml",

      "platform": "jfrog",
      "jfrogToolConfigId": "[JFROG_TOOL_ID]",
      "jfrogRepositoryName": "[JFROG_REPO]",
      "jfrogRepoType": "REPOPATHPREFIX",
      "repositoryTag": "[JFROG_URL]/[REPO]/[IMAGE]:[TAG]",

      "customImageTag": true,
      "existingContent": "image",
      "delayTime": 5,
      "kustomizeFlag": false,
      "helmFlag": false
    }
  },
  "type": ["deploy"]
}
```

**What happens**:
- Pulls approved image from JFrog Artifactory
- Updates Kubernetes deployment manifest
- Triggers ArgoCD application sync
- Deploys to Kubernetes cluster

## Complete Pipeline Template

```json
{
  "name": "[APP_NAME] GitHub Actions + ArgoCD Pipeline",
  "description": "Hybrid CI/CD with GitHub Actions, approval gates, and ArgoCD deployment",
  "active": true,
  "type": ["github_actions_pipeline"],
  "tags": [
    {"type": "pipeline", "value": "github-actions"},
    {"type": "pipeline", "value": "jfrog"},
    {"type": "pipeline", "value": "argocd"},
    {"type": "pipeline", "value": "approval-gate"}
  ],
  "workflow": [
    {
      "name": "CI-GitHub Actions",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "github_actions_workflow",
        "configuration": {
          "toolId": "[GITHUB_TOOL_ID]",
          "repositoryId": "[ORG/REPO]",
          "workflowId": "[WORKFLOW_ID]",
          "branchOrTagName": "main",
          "job": "",
          "inputs": {}
        }
      },
      "type": ["git_automation"]
    },
    {
      "name": "Approval for Prod Deploy",
      "active": true,
      "dependencies": ["CI-GitHub Actions"],
      "tool": {
        "tool_identifier": "approval",
        "configuration": {
          "contact": "[APPROVER_NAME]",
          "message": "Approve deployment to production",
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
      "type": ["interaction-gate"]
    },
    {
      "name": "ArgoCD Deploy",
      "active": true,
      "dependencies": ["Approval for Prod Deploy"],
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
          "defaultBranch": "main",
          "gitFilePath": "k8s/deployment.yaml",

          "platform": "jfrog",
          "jfrogToolConfigId": "[JFROG_TOOL_ID]",
          "jfrogRepositoryName": "[JFROG_REPO]",
          "jfrogRepoType": "REPOPATHPREFIX",
          "repositoryTag": "[JFROG_URL]/[REPO]/[IMAGE]:[TAG]",

          "customImageTag": true,
          "existingContent": "image",
          "delayTime": 5,
          "kustomizeFlag": false,
          "helmFlag": false,
          "dynamicVariables": false,
          "rollbackEnabled": false
        }
      },
      "type": ["deploy"]
    }
  ]
}
```

## GitHub Actions Workflow Example

Your GitHub Actions workflow (`.github/workflows/build.yml`):

```yaml
name: Build and Push to JFrog

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: false
        default: 'dev'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to JFrog Artifactory
        uses: docker/login-action@v2
        with:
          registry: trial0no4i8.jfrog.io
          username: ${{ secrets.JFROG_USERNAME }}
          password: ${{ secrets.JFROG_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: trial0no4i8.jfrog.io/nlp-docker/gitops-demo-app:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run tests
        run: npm test

      - name: Security scan
        run: trivy image trial0no4i8.jfrog.io/nlp-docker/gitops-demo-app:latest
```

Opsera triggers this workflow and waits for completion before proceeding to approval.

## How to Get GitHub Workflow ID

### Method 1: GitHub UI
1. Go to your repository on GitHub
2. Click **Actions** tab
3. Click on your workflow name
4. Look at the URL: `https://github.com/ORG/REPO/actions/workflows/BUILD_ID`
5. The `BUILD_ID` is your workflow ID (e.g., `216591844`)

### Method 2: GitHub API
```bash
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/ORG/REPO/actions/workflows
```

Look for the `id` field in the response.

### Method 3: GitHub CLI
```bash
gh workflow list --repo ORG/REPO
```

## JFrog Artifactory Integration

### Repository Tag Format
```
[JFROG_URL]/[REPOSITORY]/[IMAGE]:[TAG]
```

**Example**:
```
trial0no4i8.jfrog.io/nlp-docker/gitops-demo-app:latest
```

**Components**:
- **JFROG_URL**: `trial0no4i8.jfrog.io`
- **REPOSITORY**: `nlp-docker`
- **IMAGE**: `gitops-demo-app`
- **TAG**: `latest` (or version/commit SHA)

### Repository Types

| Type | Description | Use Case |
|------|-------------|----------|
| `REPOPATHPREFIX` | Path prefix format | Most common |
| `SUBDOMAIN` | Subdomain format | Legacy |
| `PORT` | Port-based routing | Rare |

## Example Request

**User**: "Create a pipeline that uses our existing GitHub Actions for builds, then deploys to Kubernetes with approval"

**Assistant Response**:
```
⚡ OPSERA │ GitHub Actions + ArgoCD Hybrid Pipeline
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

I'll create a hybrid CI/CD pipeline that:
✓ Triggers your existing GitHub Actions workflow
✓ Waits for build completion
✓ Requires manual approval
✓ Deploys with ArgoCD

Please provide:

1. GitHub Actions:
   - Repository (e.g., myorg/myapp)
   - Workflow ID (numeric, from GitHub Actions)
   - Branch to trigger

2. Artifact Storage:
   - Using JFrog Artifactory? (yes/no)
   - If yes: JFrog repository name

3. Deployment:
   - ArgoCD application name
   - Kubernetes manifest repository
   - Path to deployment YAML

4. Approval:
   - Who should approve? (name/team)
   - Custom approval message
```

## Tool Identifiers Required

| Tool | Purpose | How to Get |
|------|---------|------------|
| GitHub Tool ID | Repository & workflow access | `/api/v2/tools?type=github` |
| JFrog Tool ID | Artifactory access | `/api/v2/tools?type=jfrog` |
| ArgoCD Tool ID | GitOps deployment | `/api/v2/tools?type=argo` |

## Key Features Explained

### 1. GitHub Actions Integration

**Benefits**:
- Use existing workflows without rewriting
- Leverage GitHub Actions marketplace
- Native GitHub integration (fast, reliable)
- Keep CI logic in source repository

**How it works**:
```
Opsera → Triggers GitHub API → Workflow runs → Opsera monitors → Success/Fail
```

### 2. Approval Gates

**Benefits**:
- Production deployment control
- Compliance requirements (SOX, HIPAA)
- Change management integration
- Audit trail for deployments

**Configuration**:
```json
{
  "contact": "Release Manager",
  "message": "Production deployment requires approval",
  "sendCustomMessage": true
}
```

### 3. JFrog Artifactory

**Why JFrog?**
- Enterprise artifact management
- Universal package support
- Security scanning (Xray)
- High availability
- Access control & compliance

**Integration**:
```json
{
  "platform": "jfrog",
  "jfrogRepositoryName": "docker-local",
  "repositoryTag": "company.jfrog.io/docker-local/app:v1.0"
}
```

### 4. ArgoCD GitOps

**Benefits**:
- Declarative deployments
- Git as single source of truth
- Automatic drift detection
- Rollback capabilities

## Best Practices

### 1. Workflow Design

**Separate Concerns**:
- **GitHub Actions**: Build, test, security scanning, push artifacts
- **Opsera**: Orchestration, approvals, multi-environment promotion
- **ArgoCD**: Deployment and sync

### 2. Approval Strategy

**When to require approval**:
- ✅ Production deployments
- ✅ Infrastructure changes
- ✅ Database migrations
- ✅ Major version updates
- ❌ Dev/test environments

### 3. Tagging Strategy

**Recommended**:
- Semantic versioning: `v1.2.3`
- Commit-based: `main-abc123`
- Build number: `build-456`
- Date-based: `20251217-01`

**Avoid**:
- ❌ Always using `latest` (not traceable)
- ❌ Random tags (not sortable)

### 4. Multi-Environment

Create pipeline variants:
```
├── dev-pipeline (no approval)
├── staging-pipeline (team approval)
└── prod-pipeline (manager approval)
```

### 5. Monitoring & Notifications

Configure notifications for:
- GitHub Actions workflow failures
- Approval requests pending
- ArgoCD deployment success/failure
- Drift detection alerts

## Troubleshooting

### Issue: GitHub Actions workflow not found
**Cause**: Incorrect workflow ID or repository
**Solution**:
1. Verify workflow ID from GitHub UI or API
2. Ensure repository format is `org/repo`
3. Check GitHub tool credentials in Opsera

### Issue: Approval not triggering
**Cause**: Misconfigured approval step
**Solution**:
1. Verify `tool_identifier: "approval"`
2. Check dependencies are correct
3. Ensure `sendCustomMessage: true`

### Issue: ArgoCD can't pull from JFrog
**Cause**: Authentication or path issues
**Solution**:
1. Verify JFrog tool configuration
2. Check repository tag format
3. Ensure ArgoCD has JFrog credentials
4. Verify `platform: "jfrog"` is set

### Issue: Workflow triggers but doesn't wait
**Cause**: Opsera not monitoring completion
**Solution**:
1. Check GitHub tool permissions
2. Verify workflow dispatch is enabled
3. Review Opsera logs for API errors

## Advanced Configurations

### Parameterized GitHub Actions

Pass inputs to GitHub Actions:

```json
{
  "tool_identifier": "github_actions_workflow",
  "configuration": {
    "inputs": {
      "environment": "production",
      "version": "v1.2.3",
      "enable_tests": true
    }
  }
}
```

### Conditional Approval

Only require approval for production:

```json
{
  "tool": {
    "tool_identifier": "approval",
    "configuration": {
      "conditionalApproval": true,
      "condition": "${environment} === 'production'"
    }
  }
}
```

### Multiple ArgoCD Applications

Deploy to multiple clusters:

```json
{
  "workflow": [
    {
      "name": "Deploy to US Cluster",
      "tool": {
        "tool_identifier": "argo",
        "configuration": {
          "applicationName": "app-us-east"
        }
      }
    },
    {
      "name": "Deploy to EU Cluster",
      "tool": {
        "tool_identifier": "argo",
        "configuration": {
          "applicationName": "app-eu-west"
        }
      }
    }
  ]
}
```

### Slack Notifications

Add notifications to approval step:

```json
{
  "notification": [
    {
      "type": "slack",
      "enabled": true,
      "event": "all",
      "channel": "deployments",
      "toolId": "[SLACK_TOOL_ID]"
    }
  ]
}
```

## Integration Patterns

### Pattern 1: Multi-Stage Deployment

```
GitHub Actions (build) → Approval → ArgoCD (dev) → Approval → ArgoCD (prod)
```

### Pattern 2: Security Gate

```
GitHub Actions (build) → Grype Scan → Approval → ArgoCD (deploy)
```

### Pattern 3: Multi-Repo Orchestration

```
GitHub Actions (backend) ─┐
                          ├→ Approval → ArgoCD (deploy both)
GitHub Actions (frontend) ┘
```

## Comparison with Other Patterns

| Pattern | CI | Approval | Deploy | Best For |
|---------|----|----|--------|----------|
| **This Skill** | GitHub Actions | Opsera | ArgoCD | Teams with existing GitHub Actions |
| **Pure Opsera** | Opsera | Opsera | Opsera | Unified platform preference |
| **Jenkins + Opsera** | Jenkins | Opsera | ArgoCD | Legacy Jenkins workflows |
| **GitLab + Opsera** | GitLab CI | Opsera | ArgoCD | GitLab users |

## Security Considerations

### GitHub Actions Security

1. **Secrets Management**: Use GitHub Secrets for credentials
2. **Workflow Permissions**: Limit token permissions
3. **Third-party Actions**: Review marketplace actions before use
4. **Branch Protection**: Require reviews for workflow changes

### JFrog Security

1. **Access Control**: Use role-based access control (RBAC)
2. **Xray Scanning**: Enable vulnerability scanning
3. **Retention Policies**: Auto-delete old artifacts
4. **Audit Logs**: Monitor access and downloads

### ArgoCD Security

1. **RBAC**: Limit application access by team
2. **SSO Integration**: Use corporate identity provider
3. **Git Credentials**: Use SSH keys, not passwords
4. **Sync Policies**: Require manual sync for production

## Production Stats

```
┌─────────────────────────────────────────────────────────┐
│  PRODUCTION METRICS                                      │
├─────────────────────────────────────────────────────────┤
│  Pipeline Name: Cisco_GitHubActions_Argo_Nagalakshmi   │
│  Pipeline ID:   69425543d08886acba59140e                │
│  Deployments:   4 successful runs                       │
│  Last Status:   ✅ Success                              │
│  Trigger:       UI (manual)                             │
│  Pattern:       Production-ready                        │
└─────────────────────────────────────────────────────────┘
```

## Related Skills

- `/opsera-argo-container` - Pure ArgoCD deployments without GitHub Actions
- `/opsera-argo-kustomize-custom-tag` - Kustomize-based ArgoCD
- `/opsera-multicloud-container` - Multi-cloud with security

## When to Use This Pattern

### ✅ Use When:
- Already using GitHub Actions for CI
- Need centralized approval gates
- Want unified visibility across pipelines
- Using JFrog Artifactory
- Deploying to Kubernetes with ArgoCD
- Multiple teams with different preferences

### ❌ Don't Use When:
- Starting from scratch (use pure Opsera)
- Simple single-step deployments
- No approval requirements
- Not using Kubernetes

---

**Skill Created By**: Opsera AI Assistant
**Based On**: Production pipeline `Cisco_GitHubActions_Argo_Nagalakshmi_DND`
**Last Updated**: 2025-12-23
**Version**: 1.0
**Category**: Hybrid CI/CD, GitHub Actions, ArgoCD, JFrog

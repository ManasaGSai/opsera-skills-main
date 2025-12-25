# Opsera Argo Container Deployment Skill

> **Branding**: Follow the Opsera branding guidelines in `.claude/CLAUDE.md` for all responses.

## Description
Creates Opsera pipelines for containerized application deployments using ArgoCD with Docker build, security scanning, and multi-environment support.

## Usage
```
/opsera-argo-container [options]
```

## Parameters
- `--name`: Pipeline name (required)
- `--repo`: GitHub repository (org/repo format)
- `--image`: Docker image name
- `--registry`: Container registry (ecr | acr | gcr | dockerhub)
- `--env`: Environments to deploy (dev | prod | dev,prod)
- `--scans`: Security scans to enable (grype | sonar | fortify | blackduck | all)
- `--approval`: Require approval for prod (true | false)
- `--notifications`: slack | teams | email (comma-separated)

## Examples
```
/opsera-argo-container --name "My App" --repo "org/app" --image "myapp" --env dev
/opsera-argo-container --name "Prod App" --repo "org/app" --env dev,prod --approval true --scans all
```

---

## Pipeline Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Pre-Build      │───▶│  Docker Build   │───▶│  Security Scan  │
│  Validation     │    │  (Jenkins)      │    │  (Grype)        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                       ┌───────────────────────────────┘
                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Docker Push    │───▶│  Argo Deploy    │───▶│  Smoke Tests    │
│  (ECR)          │    │  (Dev)          │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                       ┌───────────────────────────────┘
                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Approval Gate  │───▶│  Argo Deploy    │───▶│  Release Tag    │
│  (Optional)     │    │  (Prod)         │    │  (Git)          │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

Generate unique IDs using pattern: `[6-char prefix][18-char random hex]`
Example: `676801a1b2c3d4e5f6a7b8c9`

---

## Tool Identifiers Reference

| Step Type | tool_identifier | Purpose |
|-----------|-----------------|---------|
| Pre-build script | `run_script` | Environment setup, validation |
| Docker build | `jenkins` | Build container images |
| Container scan | `grype-scan` | Vulnerability scanning |
| Docker push | `docker-push` | Push to container registry |
| ArgoCD deploy | `argo` | Kubernetes deployment |
| Smoke test | `run_script` | Post-deployment verification |
| Approval gate | `approval` | Manual approval for prod |
| Git tagging | `git-operation` | Release management |
| Code scan | `sonar` | Code quality analysis |
| Security scan | `fortify` | SAST scanning |
| Dependency scan | `blackduck` | SCA scanning |
| Secret scan | `gitscraper` | Git secrets detection |
| Parallel pipelines | `parallel-processor` | Orchestrate child pipelines |

---

## Complete Pipeline Template

### Minimal Pipeline (Dev Only)

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Argo container deployment pipeline",
  "type": ["sdlc"],
  "tags": ["argo", "container", "docker", "kubernetes"],
  "workflow": [
    {
      "_id": "676801a1b2c3d4e5f6a7b8c1",
      "name": "Pre-Build Validation",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "PreBuildScript",
          "script": "#!/bin/bash\necho \"Starting Pre-Build Checks\"\npwd && ls -la\necho \"Environment validated\"",
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
      "_id": "676801a1b2c3d4e5f6a7b8c2",
      "name": "Docker Build",
      "active": true,
      "dependencies": ["676801a1b2c3d4e5f6a7b8c1"],
      "tool": {
        "tool_identifier": "jenkins",
        "job_type": "opsera-job",
        "configuration": {
          "jobType": "BUILD",
          "buildType": "docker",
          "dockerName": "[IMAGE_NAME]",
          "dockerTagType": "timestamp",
          "dynamicTag": true,
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
          "workspaceDeleteFlag": false
        }
      },
      "type": ["build"]
    },
    {
      "_id": "676801a1b2c3d4e5f6a7b8c3",
      "name": "Grype Scan",
      "active": true,
      "dependencies": ["676801a1b2c3d4e5f6a7b8c2"],
      "tool": {
        "tool_identifier": "grype-scan",
        "configuration": {
          "jobType": "grype-scan",
          "ecrPushStepId": "676801a1b2c3d4e5f6a7b8c2",
          "clientSideThreshold": true,
          "thresholdCompliance": [
            {"level": "critical", "count": 0}
          ]
        }
      },
      "type": ["container-scan"]
    },
    {
      "_id": "676801a1b2c3d4e5f6a7b8c4",
      "name": "Docker Push",
      "active": true,
      "dependencies": ["676801a1b2c3d4e5f6a7b8c3"],
      "tool": {
        "tool_identifier": "docker-push",
        "configuration": {
          "buildType": "docker",
          "buildStepId": "676801a1b2c3d4e5f6a7b8c2",
          "awsToolConfigId": "[AWS_TOOL_ID]",
          "ecrRepoName": "[ECR_REPO_NAME]",
          "newRepo": true
        }
      },
      "type": ["publish"]
    },
    {
      "_id": "676801a1b2c3d4e5f6a7b8c5",
      "name": "Argo Deploy Dev",
      "active": true,
      "dependencies": ["676801a1b2c3d4e5f6a7b8c4"],
      "tool": {
        "tool_identifier": "argo",
        "configuration": {
          "applicationName": "[ARGO_APP_NAME]-dev",
          "toolConfigId": "[ARGO_TOOL_ID]",
          "toolUrl": "[ARGO_URL]",
          "userName": "admin",
          "gitToolId": "[GIT_TOOL_ID]",
          "gitUrl": "[REPO_URL]",
          "gitRepository": "[REPO_NAME]",
          "gitRepositoryID": "[ORG/REPO]",
          "gitBranch": "main",
          "gitFilePath": "k8s/Dev/deployment.yaml",
          "dockerStepID": "676801a1b2c3d4e5f6a7b8c4",
          "dockerStepType": "docker-push",
          "existingContent": "image",
          "type": "github",
          "delayTime": 5
        }
      },
      "type": ["deploy"],
      "notification": [
        {
          "type": "slack",
          "enabled": true,
          "event": "all",
          "channel": "[SLACK_CHANNEL]",
          "toolId": "[SLACK_TOOL_ID]"
        }
      ]
    },
    {
      "_id": "676801a1b2c3d4e5f6a7b8c6",
      "name": "Smoke Test",
      "active": true,
      "dependencies": ["676801a1b2c3d4e5f6a7b8c5"],
      "tool": {
        "tool_identifier": "run_script",
        "configuration": {
          "type": "manual_entry",
          "scriptName": "SmokeTest",
          "script": "#!/bin/bash\necho \"Running Smoke Tests\"\necho \"Checking deployed application...\"\nsleep 5\necho \"Smoke Tests Passed\"",
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

## Additional Steps (Optional)

### Approval Gate (for Production)

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8c7",
  "name": "Approval for Prod",
  "active": true,
  "dependencies": ["676801a1b2c3d4e5f6a7b8c6"],
  "tool": {
    "tool_identifier": "approval",
    "configuration": {
      "contact": "[APPROVER_EMAIL]",
      "message": "Approval required for Production deployment",
      "sendCustomMessage": true
    },
    "threshold": {
      "type": "user-approval",
      "value": {}
    }
  },
  "type": ["interaction-gate"]
}
```

### Production Argo Deployment

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8c8",
  "name": "Argo Deploy Prod",
  "active": true,
  "dependencies": ["676801a1b2c3d4e5f6a7b8c7"],
  "tool": {
    "tool_identifier": "argo",
    "configuration": {
      "applicationName": "[ARGO_APP_NAME]-prod",
      "toolConfigId": "[ARGO_TOOL_ID]",
      "toolUrl": "[ARGO_URL]",
      "userName": "admin",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitRepository": "[REPO_NAME]",
      "gitRepositoryID": "[ORG/REPO]",
      "gitBranch": "main",
      "gitFilePath": "k8s/Prod/deployment.yaml",
      "dockerStepID": "676801a1b2c3d4e5f6a7b8c4",
      "dockerStepType": "docker-push",
      "existingContent": "image",
      "type": "github",
      "delayTime": 5
    }
  },
  "type": ["deploy"]
}
```

### Release Tagging

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8c9",
  "name": "Release Tag",
  "active": true,
  "dependencies": ["676801a1b2c3d4e5f6a7b8c8"],
  "tool": {
    "tool_identifier": "git-operation",
    "configuration": {
      "action": "tag-creation",
      "tag": "v_run_count",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitBranch": "main",
      "repository": "[REPO_NAME]",
      "repoId": "[ORG/REPO]",
      "projectId": "[ORG/REPO]",
      "service": "github"
    }
  },
  "type": ["git_automation"]
}
```

### Parallel Pipeline Trigger

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8ca",
  "name": "Trigger Dependent Pipelines",
  "active": true,
  "dependencies": ["676801a1b2c3d4e5f6a7b8c6"],
  "tool": {
    "tool_identifier": "parallel-processor",
    "configuration": {
      "pipelines": ["[PIPELINE_ID_1]", "[PIPELINE_ID_2]"]
    },
    "threshold": {
      "completeFirst": true,
      "ensureSuccess": true
    }
  },
  "type": ["pipeline-orchestration"]
}
```

---

## Security Scanning Steps

### Sonar Scan (Code Quality)

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8cb",
  "name": "Sonar Scan",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "sonar",
    "job_type": "opsera-job",
    "configuration": {
      "jobType": "CODE SCAN",
      "buildToolVersion": "6.3",
      "buildType": "gradle",
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

### Git Secrets Scan

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8cc",
  "name": "Git Secrets Scan",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "gitscraper",
    "configuration": {
      "type": "git_custodian",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitBranch": "main",
      "repository": "[REPO_NAME]",
      "repoId": "[ORG/REPO]",
      "projectId": "[ORG/REPO]",
      "service": "github",
      "customEntropy": true,
      "entropy": 3.5,
      "scanEmail": true,
      "threshold": 30,
      "excludeDomains": ["example.com"]
    }
  },
  "type": ["code-scan"]
}
```

### Blackduck (Dependency Scan)

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8cd",
  "name": "Blackduck Scan",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "blackduck",
    "configuration": {
      "blackDuckToolId": "[BLACKDUCK_TOOL_ID]",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitBranch": "main",
      "gitRepository": "[REPO_NAME]",
      "gitRepositoryID": "[ORG/REPO]",
      "type": "github",
      "clientSideThreshold": true,
      "thresholdVulnerability": [
        {"level": "high", "count": 10}
      ],
      "thresholdLicence": [
        {"level": "medium", "count": 30}
      ],
      "dependencyType": [
        {"dependencyType": "java", "name": "Java openJDK Version 13", "version": "13"}
      ]
    }
  },
  "type": ["compliance-sec"]
}
```

### Fortify (SAST)

```json
{
  "_id": "676801a1b2c3d4e5f6a7b8ce",
  "name": "Fortify Scan",
  "active": true,
  "dependencies": [],
  "tool": {
    "tool_identifier": "fortify",
    "configuration": {
      "scanToolType": "Fortify On Demand",
      "toolConfigId": "[FORTIFY_TOOL_ID]",
      "gitToolId": "[GIT_TOOL_ID]",
      "gitUrl": "[REPO_URL]",
      "gitBranch": "main",
      "gitRepository": "[REPO_NAME]",
      "repoId": "[ORG/REPO]",
      "service": "github",
      "applicationName": "[APP_NAME]",
      "releaseName": "1.0",
      "assessmentType": "Static Assessment",
      "technologyStackId": "JAVA/J2EE",
      "languageLevelId": "Java 11",
      "clientSideThreshold": true,
      "thresholdVulnerability": [
        {"level": "critical", "count": 2},
        {"level": "medium", "count": 1}
      ]
    }
  },
  "type": ["compliance-sec"]
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
      "channel": "[CHANNEL_NAME]",
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
    "pipelineId": "6907e1c6dbe50ab7942d519f",
    "pipeline": {
      "_id": "6907e1c6dbe50ab7942d519f",
      "name": "speri-GOLDEN_TEMPLATES_PROD-READY-ARGO",
      "active": true,
      "type": ["sdlc"],
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
Pipeline ID: 6907e1c6dbe50ab7942d519f
Portal URL: https://portal.opsera.io/workflow/details/6907e1c6dbe50ab7942d519f/summary
```

Always include this URL in the response after successful pipeline creation.

---

## Implementation Instructions

### 1. Gather Requirements
- Pipeline name
- Source repository (GitHub)
- Docker image name
- Container registry type and credentials
- Target environments (Dev, Prod)
- ArgoCD application names
- Kubernetes manifest paths
- Required security scans
- Notification channels

### 2. Create Pipeline via API

```
mcp__opsera-ai-agent__post_api_v2_pipeline_create
```

### 3. Required Tool IDs
Before creating the pipeline, you need:
- `gitToolId`: GitHub tool configuration ID
- `toolConfigId`: Jenkins tool configuration ID
- `awsToolConfigId`: AWS credentials for ECR (if using ECR)
- `argoToolConfigId`: ArgoCD tool configuration ID
- `slackToolId`: Slack integration ID (optional)

---

## Best Practices

1. **Always include Grype scan** for container vulnerability detection
2. **Use timestamp-based Docker tags** for traceability
3. **Configure notifications** on deploy steps for visibility
4. **Add approval gates** before production deployments
5. **Use environment-specific K8s manifests** (k8s/Dev/, k8s/Prod/)
6. **Enable parallel security scans** at pipeline start
7. **Set threshold compliance** to fail on critical vulnerabilities
8. **Use git tagging** for release management

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `676801a1b2c3d4e5f6a7b8c1`

2. **"Invalid tool_identifier"**
   - Cause: Unsupported tool type
   - Fix: Use exact identifiers from the reference table above

3. **"Missing buildStepId reference"**
   - Cause: Docker push step can't find build step
   - Fix: Ensure `buildStepId` matches the Docker build step's `_id`

### Runtime Errors

1. **ArgoCD sync failure**: Verify K8s manifest path exists in repo
2. **Docker push auth error**: Check AWS/Azure credentials
3. **Grype timeout**: Increase step timeout or check image size
4. **Approval stuck**: Verify approver email is valid Opsera user

---

## Reference Pipeline

See pipeline `6907e1c6dbe50ab7942d519f` (speri-GOLDEN_TEMPLATES_PROD-READY-ARGO) for a complete working example with all step configurations.

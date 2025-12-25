# Opsera Multi-Cloud Container Deployment Skill

> **Branding**: Follow the Opsera branding guidelines in `.claude/CLAUDE.md` for all responses.

## Description
Creates Opsera pipelines for multi-cloud container deployments with comprehensive security guardrails. Deploys to AWS (EKS/ECR), Azure (AKS/ACR), and GCP (GKE/GCR) with full security scanning and environment promotion gates.

## Usage
```
/opsera-multicloud-container [options]
```

## Parameters
- `--name`: Pipeline name (required)
- `--repo`: GitHub repository (org/repo format)
- `--image`: Docker image name
- `--clouds`: aws | azure | gcp | all (default: all)
- `--envs`: dev | staging | prod | dev,staging,prod (default: dev,staging,prod)
- `--security`: full | essential | minimal (default: full)
- `--approval`: Require approval gates (true | false, default: true)
- `--notifications`: slack | teams | email (comma-separated)

## Security Scan Levels
- `full`: SonarQube + Grype + BlackDuck + Twistlock
- `essential`: SonarQube + Grype
- `minimal`: Grype only

## Examples
```
/opsera-multicloud-container --name "My App" --repo "org/app" --clouds all --envs dev,staging,prod
/opsera-multicloud-container --name "API Service" --clouds aws,azure --security full --approval true
/opsera-multicloud-container --name "Dev Only" --clouds aws --envs dev --security minimal
```

---

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              GIT CHECKOUT                                    │
│                            (GitHub: main branch)                             │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
     ┌────────────────┐              ┌────────────────┐
     │  Docker Build  │              │   SonarQube    │
     │                │              │   Analysis     │
     └───────┬────────┘              └───────┬────────┘
             │                               │
     ┌───────┴───────────────────────────────┤
     ▼               ▼               ▼       │
┌─────────┐   ┌───────────┐   ┌──────────┐   │
│  Grype  │   │ BlackDuck │   │ Twistlock│   │
│  Scan   │   │   Scan    │   │   Scan   │   │
└────┬────┘   └─────┬─────┘   └────┬─────┘   │
     └──────────────┼──────────────┴─────────┘
                    │ (All security gates must pass)
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
┌─────────┐   ┌──────────┐   ┌─────────┐
│ AWS ECR │   │ Azure ACR│   │ GCP GCR │
│  Push   │   │   Push   │   │  Push   │
└────┬────┘   └────┬─────┘   └────┬────┘
     ▼              ▼              ▼
┌─────────┐   ┌──────────┐   ┌─────────┐
│ AWS EKS │   │Azure AKS │   │ GCP GKE │
│  (Dev)  │   │  (Dev)   │   │  (Dev)  │
└────┬────┘   └────┬─────┘   └────┬────┘
     └──────────────┼──────────────┘
                    ▼
          ┌─────────────────┐
          │ STAGING APPROVAL│ ← Lead Engineer
          └────────┬────────┘
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
┌─────────┐  ┌──────────┐  ┌─────────┐
│ AWS EKS │  │Azure AKS │  │ GCP GKE │
│(Staging)│  │(Staging) │  │(Staging)│
└────┬────┘  └────┬─────┘  └────┬────┘
     └─────────────┼─────────────┘
                   ▼
         ┌──────────────────┐
         │  PROD APPROVAL   │ ← Release Manager + Security Team
         └────────┬─────────┘
     ┌────────────┼────────────┐
     ▼            ▼            ▼
┌─────────┐ ┌──────────┐ ┌─────────┐
│ AWS EKS │ │Azure AKS │ │ GCP GKE │
│ (Prod)  │ │  (Prod)  │ │ (Prod)  │
└─────────┘ └──────────┘ └─────────┘
```

---

## Step ID Requirements

**CRITICAL**: Each workflow step `_id` MUST be a valid 24-character hexadecimal string (MongoDB ObjectId format).

Generate unique IDs using pattern: `[6-char prefix][18-char random hex]`
Example: `6a4b5c6d7e8f9a0b1c2d3e01`

---

## Tool Identifiers Reference

| Step Type | tool_identifier | Purpose |
|-----------|-----------------|---------|
| Git checkout | `github` | Clone source repository |
| Docker build | `docker` | Build container image |
| Code quality | `sonarqube` | Code quality analysis |
| Container scan | `grype` | Vulnerability scanning |
| Dependency scan | `blackduck` | Open source security |
| Compliance scan | `twistlock` | Container compliance |
| AWS ECR push | `aws_ecr` | Push to Amazon ECR |
| Azure ACR push | `azure_acr` | Push to Azure ACR |
| GCP GCR push | `gcp_gcr` | Push to Google GCR |
| AWS EKS deploy | `aws_eks` | Deploy to Amazon EKS |
| Azure AKS deploy | `azure_aks` | Deploy to Azure AKS |
| GCP GKE deploy | `gcp_gke` | Deploy to Google GKE |
| Approval gate | `approval` | Manual approval |

---

## Security Guardrails Configuration

### SonarQube (Code Quality)
```json
{
  "tool_identifier": "sonarqube",
  "configuration": {
    "project_key": "[PROJECT_KEY]",
    "quality_gate": true,
    "fail_on_quality_gate": true
  }
}
```

### Grype (Container Vulnerabilities)
```json
{
  "tool_identifier": "grype",
  "configuration": {
    "severity_threshold": "high",
    "fail_on_severity": true,
    "ignore_unfixed": false
  }
}
```

### BlackDuck (Open Source Security)
```json
{
  "tool_identifier": "blackduck",
  "configuration": {
    "scan_mode": "full",
    "policy_check": true,
    "fail_on_policy_violation": true
  }
}
```

### Twistlock (Container Compliance)
```json
{
  "tool_identifier": "twistlock",
  "configuration": {
    "vulnerability_threshold": "high",
    "compliance_check": true,
    "fail_on_critical": true
  }
}
```

---

## Complete Pipeline Template

### Full Multi-Cloud Pipeline (Dev → Staging → Prod)

```json
{
  "name": "[PIPELINE_NAME]",
  "description": "Multi-cloud container deployment pipeline with full security guardrails. Deploys to AWS (EKS/ECR), Azure (AKS/ACR), and GCP (GKE/GCR) with SonarQube, Grype, BlackDuck, and Twistlock security scanning. Includes Dev → Staging → Prod promotion with approval gates.",
  "type": ["containerization", "security", "multi-cloud"],
  "tags": [
    {"type": "deployment", "value": "multi-cloud"},
    {"type": "security", "value": "full-suite"},
    {"type": "environment", "value": "dev-staging-prod"}
  ],
  "active": true,
  "workflow": [
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e01",
      "name": "Git Checkout",
      "active": true,
      "dependencies": [],
      "tool": {
        "tool_identifier": "github",
        "configuration": {
          "repository": "[ORG/REPO]",
          "branch": "main",
          "workspace": "/workspace"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e02",
      "name": "Docker Build",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e01"],
      "tool": {
        "tool_identifier": "docker",
        "configuration": {
          "dockerfile": "Dockerfile",
          "context": ".",
          "image_name": "[IMAGE_NAME]"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e03",
      "name": "SonarQube Analysis",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e01"],
      "tool": {
        "tool_identifier": "sonarqube",
        "configuration": {
          "project_key": "[PROJECT_KEY]",
          "quality_gate": true,
          "fail_on_quality_gate": true
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e04",
      "name": "Grype Container Scan",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e02"],
      "tool": {
        "tool_identifier": "grype",
        "configuration": {
          "severity_threshold": "high",
          "fail_on_severity": true,
          "ignore_unfixed": false
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e05",
      "name": "BlackDuck Scan",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e02"],
      "tool": {
        "tool_identifier": "blackduck",
        "configuration": {
          "scan_mode": "full",
          "policy_check": true,
          "fail_on_policy_violation": true
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e06",
      "name": "Twistlock Scan",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e02"],
      "tool": {
        "tool_identifier": "twistlock",
        "configuration": {
          "vulnerability_threshold": "high",
          "compliance_check": true,
          "fail_on_critical": true
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e07",
      "name": "Push to AWS ECR",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e03", "6a4b5c6d7e8f9a0b1c2d3e04", "6a4b5c6d7e8f9a0b1c2d3e05", "6a4b5c6d7e8f9a0b1c2d3e06"],
      "tool": {
        "tool_identifier": "aws_ecr",
        "configuration": {
          "region": "[AWS_REGION]",
          "repository": "[ECR_REPO]",
          "tag": "${BUILD_NUMBER}"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e08",
      "name": "Push to Azure ACR",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e03", "6a4b5c6d7e8f9a0b1c2d3e04", "6a4b5c6d7e8f9a0b1c2d3e05", "6a4b5c6d7e8f9a0b1c2d3e06"],
      "tool": {
        "tool_identifier": "azure_acr",
        "configuration": {
          "registry": "[ACR_REGISTRY].azurecr.io",
          "repository": "[ACR_REPO]",
          "tag": "${BUILD_NUMBER}"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e09",
      "name": "Push to GCP GCR",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e03", "6a4b5c6d7e8f9a0b1c2d3e04", "6a4b5c6d7e8f9a0b1c2d3e05", "6a4b5c6d7e8f9a0b1c2d3e06"],
      "tool": {
        "tool_identifier": "gcp_gcr",
        "configuration": {
          "project": "[GCP_PROJECT]",
          "repository": "[GCR_REPO]",
          "tag": "${BUILD_NUMBER}"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e10",
      "name": "Deploy to AWS EKS (Dev)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e07"],
      "tool": {
        "tool_identifier": "aws_eks",
        "configuration": {
          "cluster": "[DEV_EKS_CLUSTER]",
          "namespace": "[APP_NAMESPACE]-dev",
          "manifest": "k8s/dev/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e11",
      "name": "Deploy to Azure AKS (Dev)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e08"],
      "tool": {
        "tool_identifier": "azure_aks",
        "configuration": {
          "cluster": "[DEV_AKS_CLUSTER]",
          "resource_group": "[DEV_RESOURCE_GROUP]",
          "namespace": "[APP_NAMESPACE]-dev",
          "manifest": "k8s/dev/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e12",
      "name": "Deploy to GCP GKE (Dev)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e09"],
      "tool": {
        "tool_identifier": "gcp_gke",
        "configuration": {
          "cluster": "[DEV_GKE_CLUSTER]",
          "zone": "[GCP_ZONE]",
          "namespace": "[APP_NAMESPACE]-dev",
          "manifest": "k8s/dev/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e13",
      "name": "Staging Approval Gate",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e10", "6a4b5c6d7e8f9a0b1c2d3e11", "6a4b5c6d7e8f9a0b1c2d3e12"],
      "tool": {
        "tool_identifier": "approval",
        "configuration": {
          "approvers": ["[LEAD_ENGINEER_EMAIL]"],
          "timeout_hours": 24,
          "message": "Approve deployment to Staging environment?"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e14",
      "name": "Deploy to AWS EKS (Staging)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e13"],
      "tool": {
        "tool_identifier": "aws_eks",
        "configuration": {
          "cluster": "[STAGING_EKS_CLUSTER]",
          "namespace": "[APP_NAMESPACE]-staging",
          "manifest": "k8s/staging/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e15",
      "name": "Deploy to Azure AKS (Staging)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e13"],
      "tool": {
        "tool_identifier": "azure_aks",
        "configuration": {
          "cluster": "[STAGING_AKS_CLUSTER]",
          "resource_group": "[STAGING_RESOURCE_GROUP]",
          "namespace": "[APP_NAMESPACE]-staging",
          "manifest": "k8s/staging/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e16",
      "name": "Deploy to GCP GKE (Staging)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e13"],
      "tool": {
        "tool_identifier": "gcp_gke",
        "configuration": {
          "cluster": "[STAGING_GKE_CLUSTER]",
          "zone": "[GCP_ZONE]",
          "namespace": "[APP_NAMESPACE]-staging",
          "manifest": "k8s/staging/deployment.yaml"
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e17",
      "name": "Production Approval Gate",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e14", "6a4b5c6d7e8f9a0b1c2d3e15", "6a4b5c6d7e8f9a0b1c2d3e16"],
      "tool": {
        "tool_identifier": "approval",
        "configuration": {
          "approvers": ["[RELEASE_MANAGER_EMAIL]", "[SECURITY_TEAM_EMAIL]"],
          "timeout_hours": 48,
          "require_all_approvers": true,
          "message": "Approve deployment to PRODUCTION? Requires security team sign-off."
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e18",
      "name": "Deploy to AWS EKS (Prod)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e17"],
      "tool": {
        "tool_identifier": "aws_eks",
        "configuration": {
          "cluster": "[PROD_EKS_CLUSTER]",
          "namespace": "[APP_NAMESPACE]-prod",
          "manifest": "k8s/prod/deployment.yaml",
          "rolling_update": true
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e19",
      "name": "Deploy to Azure AKS (Prod)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e17"],
      "tool": {
        "tool_identifier": "azure_aks",
        "configuration": {
          "cluster": "[PROD_AKS_CLUSTER]",
          "resource_group": "[PROD_RESOURCE_GROUP]",
          "namespace": "[APP_NAMESPACE]-prod",
          "manifest": "k8s/prod/deployment.yaml",
          "rolling_update": true
        }
      }
    },
    {
      "_id": "6a4b5c6d7e8f9a0b1c2d3e20",
      "name": "Deploy to GCP GKE (Prod)",
      "active": true,
      "dependencies": ["6a4b5c6d7e8f9a0b1c2d3e17"],
      "tool": {
        "tool_identifier": "gcp_gke",
        "configuration": {
          "cluster": "[PROD_GKE_CLUSTER]",
          "zone": "[GCP_ZONE]",
          "namespace": "[APP_NAMESPACE]-prod",
          "manifest": "k8s/prod/deployment.yaml",
          "rolling_update": true
        }
      }
    }
  ]
}
```

---

## Approval Gate Configuration

### Staging Approval
```json
{
  "tool_identifier": "approval",
  "configuration": {
    "approvers": ["lead-engineer@company.com"],
    "timeout_hours": 24,
    "message": "Approve deployment to Staging environment?"
  }
}
```

### Production Approval (Multi-Approver)
```json
{
  "tool_identifier": "approval",
  "configuration": {
    "approvers": ["release-manager@company.com", "security-team@company.com"],
    "timeout_hours": 48,
    "require_all_approvers": true,
    "message": "Approve deployment to PRODUCTION? Requires security team sign-off."
  }
}
```

---

## API Response Structure

On successful pipeline creation:

```json
{
  "status": "success",
  "data": {
    "pipelineId": "694a6eb52c0fedbfcf51180d",
    "dependencyResolution": {
      "dependenciesResolved": 32,
      "stepsProcessed": 20
    },
    "pipeline": {
      "_id": "694a6eb52c0fedbfcf51180d",
      "name": "MultiCloud Container Deployment with Security",
      "active": true,
      "type": ["containerization", "security", "multi-cloud"],
      "workflow": {
        "run_count": 0,
        "plan": [...]
      }
    }
  },
  "metadata": {
    "message": "Pipeline created successfully",
    "apiVersion": "v2"
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
Pipeline ID: 694a6eb52c0fedbfcf51180d
Portal URL: https://portal.opsera.io/workflow/details/694a6eb52c0fedbfcf51180d/summary
```

Always include this URL in the response after successful pipeline creation.

---

## Implementation Instructions

### 1. Gather Requirements
- Pipeline name
- Source repository (GitHub)
- Docker image name
- Target cloud providers (AWS, Azure, GCP)
- Target environments (Dev, Staging, Prod)
- Security scan level (full, essential, minimal)
- Approval requirements
- Notification channels

### 2. Create Pipeline via API

```
mcp__opsera-ai-agent__post_api_v2_pipeline_create
```

### 3. Required Tool/Credential IDs
Before creating the pipeline, you need:
- `gitToolId`: GitHub tool configuration ID
- AWS credentials for ECR/EKS
- Azure credentials for ACR/AKS
- GCP credentials for GCR/GKE
- Security tool configurations (SonarQube, BlackDuck, Twistlock)

### 4. Kubernetes Manifests
Create environment-specific manifests in your repository:
```
k8s/
├── dev/
│   └── deployment.yaml
├── staging/
│   └── deployment.yaml
└── prod/
    └── deployment.yaml
```

---

## Best Practices

1. **Security First**: Always include at least Grype scan for container vulnerabilities
2. **Multi-Approver for Prod**: Require both release manager and security team approval
3. **Rolling Updates**: Enable rolling updates for production deployments
4. **Environment Isolation**: Use separate namespaces per environment
5. **Consistent Tagging**: Use build numbers for container image tags
6. **Parallel Security Scans**: Run all security scans in parallel after build
7. **Fail Fast**: Configure security gates to fail on high/critical findings

---

## Troubleshooting

### API Validation Errors

1. **"Step 'X' is missing a valid _id"**
   - Cause: Step `_id` is not a valid 24-character hex string
   - Fix: Use format like `6a4b5c6d7e8f9a0b1c2d3e01`

2. **"Invalid dependencies"**
   - Cause: Dependency references non-existent step ID
   - Fix: Ensure all dependency IDs match actual step `_id` values

3. **"Pipeline plan structure validation failed"**
   - Cause: Invalid workflow structure
   - Fix: Verify each step has required fields: `_id`, `name`, `active`, `tool`

### Runtime Issues

1. **Security scan timeout**: Increase step timeout for large images
2. **Registry push auth failure**: Verify cloud credentials are configured
3. **Kubernetes deploy failure**: Check manifest paths and cluster connectivity
4. **Approval stuck**: Verify approver emails are valid Opsera users

---

## Reference Pipeline

See pipeline `694a6eb52c0fedbfcf51180d` for a complete working example with all 20 steps configured for multi-cloud deployment with full security guardrails.

Portal URL: https://portal.opsera.io/workflow/details/694a6eb52c0fedbfcf51180d/summary

# Opsera Skills

Claude Code skills for Opsera pipeline automation.

## Available Skills

### /opsera-terraform

Creates Opsera pipelines for Terraform infrastructure-as-code deployments.

**Supported Cloud Providers:**
- AWS (S3 backend)
- Azure (AZURERM backend)
- GCP (GCS backend)

**Usage:**
```
/opsera-terraform aws --name "My Pipeline" --repo "org/repo"
/opsera-terraform azure --name "Azure Infra" --path "terraform/azure"
```

See [.claude/skills/opsera-terraform.md](.claude/skills/opsera-terraform.md) for full documentation.

---

### /opsera-argo-container

Creates Opsera pipelines for containerized application deployments using ArgoCD.

**Features:**
- Docker build and push (ECR, ACR, GCR)
- Security scanning (Grype, Sonar, Fortify, Blackduck)
- Multi-environment deployments (Dev, Prod)
- Approval gates for production
- Smoke tests and parallel pipeline orchestration

**Usage:**
```
/opsera-argo-container --name "My App" --repo "org/app" --image "myapp" --env dev
/opsera-argo-container --name "Prod App" --repo "org/app" --env dev,prod --approval true
```

See [.claude/skills/opsera-argo-container.md](.claude/skills/opsera-argo-container.md) for full documentation.

---

### /opsera-azure-functions

Creates Opsera pipelines for Azure Functions ZIP deployments.

**Features:**
- Build and package (Maven, Gradle, npm, pip, dotnet)
- Push to Azure Blob Storage
- ZIP deployment to Azure Functions
- Multi-runtime support (Java, Node.js, Python, .NET)

**Usage:**
```
/opsera-azure-functions --name "My Function" --repo "org/azure-func" --runtime java
/opsera-azure-functions --name "API" --function-app "myapp-func" --resource-group "myapp-rg"
```

See [.claude/skills/opsera-azure-functions.md](.claude/skills/opsera-azure-functions.md) for full documentation.

---

### /opsera-azure-webapp

Creates Opsera pipelines for Azure Web App package deployments.

**Features:**
- Build and package (npm, Maven, Gradle, pip, dotnet)
- Push to Azure Blob Storage
- Package (ZIP) deployment to Azure Web Apps
- Multi-runtime support (Node.js, Java, Python, .NET)
- App settings and connection strings configuration
- Multi-environment deployments with approval gates

**Usage:**
```
/opsera-azure-webapp --name "My Web App" --repo "org/webapp" --runtime node
/opsera-azure-webapp --name "API Service" --webapp-name "myapi-app" --resource-group "myapp-rg"
```

See [.claude/skills/opsera-azure-webapp.md](.claude/skills/opsera-azure-webapp.md) for full documentation.

---

### /opsera-multicloud-container

Creates Opsera pipelines for multi-cloud container deployments with comprehensive security guardrails.

**Features:**
- Multi-cloud deployment (AWS EKS, Azure AKS, GCP GKE)
- Multi-registry support (ECR, ACR, GCR)
- Full security scanning (SonarQube, Grype, BlackDuck, Twistlock)
- Environment promotion (Dev → Staging → Prod)
- Multi-approver gates for production
- Rolling updates for zero-downtime deployments

**Usage:**
```
/opsera-multicloud-container --name "My App" --clouds all --envs dev,staging,prod --security full
/opsera-multicloud-container --name "API Service" --clouds aws,azure --approval true
```

See [.claude/skills/opsera-multicloud-container.md](.claude/skills/opsera-multicloud-container.md) for full documentation.

---

### /opsera-aws-lambda

Creates Opsera pipelines for AWS Lambda serverless function deployments.

**Features:**
- Build automation (Maven, Gradle, npm, pip, dotnet)
- S3 artifact storage
- Lambda function deployment (create/update)
- Security scanning (Veracode, Sonar)
- Multi-runtime support (Java, Node.js, Python, .NET)
- Smoke testing and validation

**Usage:**
```
/opsera-aws-lambda --name "My Lambda" --repo "org/lambda-func" --runtime java
/opsera-aws-lambda --name "API Lambda" --function-name "my-api-handler" --s3-bucket "my-artifacts"
/opsera-aws-lambda --name "Python Lambda" --runtime python --security-scan sonar
```

See [.claude/skills/opsera-aws-lambda.md](.claude/skills/opsera-aws-lambda.md) for full documentation.

---

## Installation

Add this repository as a skill source in your Claude Code configuration.

# Pipeline Examples

This directory contains comprehensive example pipeline configurations organized by pipeline type and complexity level. Each example is fully functional and demonstrates specific features of the Choreo Pipeline Specification.

## Quick Start Guide

### New to Choreo Pipelines?
1. **Start Simple**: [Hello World Build Pipeline](./build/hello-world.yaml) - Your first pipeline in 5 minutes
2. **Add Testing**: [Node.js with Tests](./build/nodejs-app.yaml) - Build, test, and validate  
3. **Deploy Apps**: [Multi-Environment Deployment](./deployment/multi-environment-deployment.yaml) - Deploy across environments
4. **Use Templates**: [Choreo Templates Guide](./choreo-templates/README.md) - Leverage pre-built components

### Learning Path by Experience Level

#### **Beginner** (New to Pipelines)
Start here if you're new to pipelines or build/deployment concepts:

| Example | Description | Time | Concepts |
|---------|-------------|------|----------|
| [Hello World](./build/hello-world.yaml) | Simplest possible pipeline | 2 min | Basic structure, steps |
| [Simple Build](./build/simple-build.yaml) | Build and test a simple app | 5 min | Templates, inline scripts |
| [Environment Variables](./build/environment-variables.yaml) | Using secrets and variables | 5 min | Configuration management |

#### **Intermediate** (Some Pipeline experience)
Build on basics with real-world patterns:

| Example | Description | Time | Concepts |
|---------|-------------|------|----------|
| [Node.js Complete](./build/nodejs-app.yaml) | Full Node.js build pipeline | 10 min | Caching, testing, artifacts |
| [Docker Multi-stage](./build/docker-multistage.yaml) | Efficient Docker builds | 10 min | Multi-stage builds, optimization |
| [Parallel Testing](./build/parallel-execution.yaml) | Speed up with parallelization | 10 min | Parallel execution, matrix builds |
| [Security Integration](./build/security-pipeline.yaml) | Built-in security scanning | 15 min | Security templates, gates |

#### **Advanced** (Pipeline Expert)
Complex scenarios and enterprise patterns:

| Example | Description | Time | Concepts |
|---------|-------------|------|----------|
| [Microservices Build](./build/microservices-monorepo.yaml) | Monorepo with multiple services | 20 min | Conditional execution, path filtering |
| [Cross-Platform Builds](./build/cross-platform.yaml) | Build for multiple architectures | 15 min | Matrix builds, multi-arch Docker |
| [Enterprise Security](./build/enterprise-security.yaml) | Comprehensive security pipeline | 25 min | Multiple scanners, compliance reporting |

## Pipeline Types

### [Build Pipelines](./build/) - Fast Feedback
**Perfect for**: Code validation, testing, building artifacts
**Constraints**: 60-minute timeout, 8Gi memory, 4 CPU cores
**Features**: Automatic source checkout, built-in artifact management, optimized for speed

| Example | Language/Stack | Complexity | Features Demonstrated |
|---------|---------------|------------|----------------------|
| [hello-world.yaml](./build/hello-world.yaml) | Any | Beginner | Basic pipeline structure |
| [nodejs-app.yaml](./build/nodejs-app.yaml) | Node.js | Intermediate | Testing, caching, artifacts |
| [python-ml.yaml](./build/python-ml.yaml) | Python | Intermediate | ML pipelines, data validation |
| [java-maven.yaml](./build/java-maven.yaml) | Java/Maven | Intermediate | Maven builds, JUnit testing |
| [go-microservice.yaml](./build/go-microservice.yaml) | Go | Intermediate | Go modules, cross-compilation |
| [docker-multistage.yaml](./build/docker-multistage.yaml) | Docker | Advanced | Multi-stage builds, optimization |
| [monorepo-nx.yaml](./build/monorepo-nx.yaml) | TypeScript/Nx | Advanced | Monorepo, affected builds |

### [CD Pipelines](./cd/) - Deployment (Environment-Aware)
**Perfect for**: Application deployment, release management
**Constraints**: 120-minute timeout, environment controls
**Features**: Progressive deployments, approval gates, rollback strategies

| Example | Deployment Type | Complexity | Features Demonstrated |
|---------|----------------|------------|----------------------|
| [simple-deployment.yaml](./cd/simple-deployment.yaml) | Basic | Beginner | Single environment deployment |
| [multi-environment.yaml](./cd/multi-environment-deployment.yaml) | Progressive | Intermediate | Dev→Staging→Prod progression |
| [canary-deployment.yaml](./cd/canary-deployment.yaml) | Canary | Advanced | Traffic splitting, monitoring |
| [blue-green.yaml](./cd/blue-green-deployment.yaml) | Blue-Green | Advanced | Zero-downtime deployments |
| [kubernetes-helm.yaml](./cd/kubernetes-helm.yaml) | Kubernetes | Advanced | Helm charts, K8s deployments |
| [approval-gates.yaml](./cd/approval-gates.yaml) | Gated | Intermediate | Manual approvals, stakeholder sign-off |

### [Automation Pipelines](./automation/) - Workflows (Unlimited)
**Perfect for**: Data processing, scheduled tasks, infrastructure automation
**Constraints**: None - unlimited resources and duration
**Features**: Cron scheduling, persistent storage, long-running operations

| Example | Use Case | Complexity | Features Demonstrated |
|---------|----------|------------|----------------------|
| [data-processing.yaml](./automation/data-processing.yaml) | ETL | Intermediate | Data pipelines, transformations |
| [scheduled-backups.yaml](./automation/scheduled-tasks.yaml) | Maintenance | Beginner | Cron triggers, file operations |
| [infrastructure-provisioning.yaml](./automation/infrastructure-automation.yaml) | IaC | Advanced | Terraform, cloud provisioning |
| [monitoring-alerts.yaml](./automation/monitoring-alerts.yaml) | Operations | Intermediate | Health checks, alerting |
| [report-generation.yaml](./automation/report-generation.yaml) | Business | Intermediate | Data aggregation, PDF reports |

### [Choreo Templates](./choreo-templates/) - Pre-built Components
**Perfect for**: Learning template usage, quick implementation
**Focus**: Template-driven development with minimal custom code

| Example | Category | Templates Used | Description |
|---------|----------|---------------|-------------|
| [build-templates.yaml](./choreo-templates/build-templates.yaml) | Build | buildpack-build, docker-build, npm-build | Complete build workflows |
| [security-templates.yaml](./choreo-templates/security-templates.yaml) | Security | trivy-scan, checkov-scan, sast-scan | Security-first pipelines |
| [deployment-templates.yaml](./choreo-templates/deployment-templates.yaml) | Deploy | deploy, helm-deploy, canary-deploy | Advanced deployment strategies |
| [testing-templates.yaml](./choreo-templates/testing-templates.yaml) | Testing | test-runner, e2e-test, performance-test | Comprehensive testing |

## Learning Scenarios

### Real-World Use Cases

#### Startup to Scale
1. **MVP Development** → [Simple Node.js Build](./build/nodejs-app.yaml)
2. **Add Security** → [Security Integration](./build/security-pipeline.yaml)
3. **Deploy to Production** → [Multi-Environment CD](./cd/multi-environment-deployment.yaml)
4. **Scale Operations** → [Automated Monitoring](./automation/monitoring-alerts.yaml)

#### Enterprise Migration
1. **Proof of Concept** → [Hello World](./build/hello-world.yaml)
2. **Legacy Integration** → [Enterprise Security](./build/enterprise-security.yaml)
3. **Multi-Service Architecture** → [Microservices Pipeline](./build/microservices-monorepo.yaml)
4. **Production Deployment** → [Blue-Green with Approval](./cd/blue-green-deployment.yaml)

#### Platform Team
1. **Template Development** → [Choreo Templates Guide](./choreo-templates/README.md)
2. **Standardization** → [Template-driven Builds](./choreo-templates/build-templates.yaml)
3. **Governance** → [Security Templates](./choreo-templates/security-templates.yaml)
4. **Self-Service** → [Complete Template Examples](./choreo-templates/)

## Implementation Guide

### Step 1: Choose Your Example
```bash
# Browse by pipeline type
ls examples/build/          # For CI pipelines
ls examples/cd/             # For deployment pipelines  
ls examples/automation/     # For automation workflows
ls examples/choreo-templates/ # For template usage
```

### Step 2: Copy and Customize
```bash
# Copy the YAML content from examples/build/nodejs-app.yaml
# Paste it into Choreo Console pipeline configuration

# Customize for your needs:
# - Replace placeholder values
# - Update environment variables  
# - Modify steps as needed
```

### Step 3: Configure Variables
```yaml
# Set in Choreo Console
VARIABLES:
  APP_NAME: "my-application"
  VERSION: "1.0.0"
  ENVIRONMENT: "staging"

SECRETS:
  API_KEY: "your-secret-api-key"
  DATABASE_URL: "postgresql://..."
```

### Step 4: Test and Deploy
1. **Commit and push** your pipeline configuration
2. **Monitor execution** in Choreo Console
3. **Review logs** and optimize as needed
4. **Iterate** based on results

## Comparison Matrix

### Pipeline Types Feature Comparison

| Feature | Build (CI) | CD | Automation |
|---------|------------|----|-----------| 
| **Timeout** | 60 minutes | 120 minutes | Unlimited |
| **Resources** | 8Gi / 4 CPU | 16Gi / 8 CPU | Unlimited |
| **Source Checkout** | ✅ Automatic | ❌ Manual | ❌ Manual |
| **Artifacts** | ✅ Built-in | ⚠️ Manual | ⚠️ Manual |
| **Caching** | ✅ Optimized | ⚠️ Manual | ⚠️ Manual |
| **Scheduling** | ❌ Code-driven | ❌ Manual | ✅ Cron |
| **Approvals** | ❌ No | ✅ Yes | ⚠️ Custom |
| **Environments** | ❌ No | ✅ Yes | ❌ No |
| **Best For** | Fast feedback | Deployments | Long tasks |

  ### Template Optimization by Pipeline Type

| Template Category | Build (CI) | CD | Automation |
|-------------------|------------|----|-----------| 
| **🏗️ Build** | ✅ Optimized | ⚠️ Limited | ⚠️ Basic |
| **🔒 Security** | ✅ Optimized | ⚠️ Limited | ⚠️ Basic |
| **🧪 Testing** | ✅ Optimized | ⚠️ Limited | ⚠️ Basic |
| **🚀 Deployment** | ⚠️ Basic | ✅ Optimized | ⚠️ Custom |
| **📢 Notifications** | ✅ Good | ✅ Good | ✅ Good |
| **🔧 Utilities** | ✅ Good | ✅ Good | ✅ Good |

## Contributing Examples

### Adding New Examples
1. **Choose the right category** based on pipeline type and complexity
2. **Create descriptive filename** using kebab-case (e.g., `nodejs-express-api.yaml`)
3. **Add comprehensive comments** explaining each section
4. **Include realistic configuration** with placeholder values
5. **Test thoroughly** before submitting
6. **Update this README** with your example

### Example Template
```yaml
# Brief description of what this pipeline does
# Use case: When to use this example
# Prerequisites: What's needed to run this

pipelineType: ci  # or cd, automation
version: "1.0"

# Explain the purpose of these parameters
arguments:
  parameters:
    - name: app_name
      value: "{{VARIABLES.APP_NAME}}"  # Set in Choreo Console

steps:
  # Step 1: Explain what this step does
  - name: Descriptive Step Name
    template: choreo/relevant-template@v1
    arguments:
      parameters:
        - name: key_parameter
          value: "example_value"
    
  # Step 2: Show inline script usage
  - name: Custom Logic
    inlineScript: |
      #!/bin/bash
      # Explain the custom logic here
      echo "Processing..."
```

### Quality Checklist
- [ ] **Clear purpose**: Example has obvious use case
- [ ] **Proper comments**: Each section is documented
- [ ] **Realistic values**: Uses believable configuration
- [ ] **Error handling**: Includes retry strategies where appropriate
- [ ] **Security**: Follows security best practices
- [ ] **Performance**: Optimized for pipeline type
- [ ] **Maintainable**: Easy to understand and modify

## Getting Help

- **📖 Documentation**: [Core Concepts](../docs/specification/concepts.md)
- **🎯 API Reference**: [Complete API](../docs/api-reference/index.md)
- **🏗️ Templates**: [Choreo Templates Guide](../docs/choreo-templates/overview.md)
- **🔄 Migration**: [Platform Migration Guides](../docs/guides/migration.md)
- **❓ Troubleshooting**: [Common Issues](../docs/guides/troubleshooting.md)

---

**💡 Tip**: Start with simple examples and gradually work your way up to more complex scenarios. Each example builds on concepts from simpler ones!
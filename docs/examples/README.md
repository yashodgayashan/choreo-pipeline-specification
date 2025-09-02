# Pipeline Examples

This directory contains focused, production-ready pipeline examples demonstrating key build patterns and advanced features of the Choreo Pipeline Specification.

## Build Pipeline Examples

Three comprehensive examples covering the most common build scenarios:

### **Language-Specific Builds**

| Example | Language/Stack | Key Features | Best For |
|---------|---------------|-------------|----------|
| [Go Build](./build/go-build.md) | Go | Container sets, parallel execution, cross-platform builds | Microservices, CLI tools |
| [Docker Build](./build/docker-build.md) | Docker | Multi-stage builds, security scanning, retries & timeouts | Containerized applications |
| [Java Build](./build/java-build.md) | Java/Maven | Comprehensive variables & secrets, database integration | Enterprise Spring Boot apps |

### **Advanced Features Demonstrated**

#### **Parallel Execution & Container Sets**
```yaml
# Example from Go Build
- - name: Unit Tests
    containerSet:
      - name: test-runner
        image: golang:1.21-alpine
  - name: Integration Tests
    containerSet:
      - name: integration-test
        image: golang:1.21-alpine
```

#### **Retries & Timeouts**
```yaml
# Example from Docker Build
timeout: 1200       # 20 minute timeout
retries: 3          # Retry up to 3 times
```

#### **Variables & Secrets Management**
```yaml
# Example from Java Build
env:
  - name: DATABASE_URL
    value: "{{SECRETS.TEST_DATABASE_URL}}"
  - name: BUILD_PROFILE
    value: "{{VARIABLES.BUILD_PROFILE}}"
```

## Pipeline Types

### [Build Pipelines](./build/) - CI/CD Build & Test
**Perfect for**: Code validation, testing, building artifacts, security scanning
**Constraints**: 60-minute timeout, 8Gi memory, 4 CPU cores
**Features**: Automatic source checkout, built-in artifact management, parallel execution
### [Automation Pipelines](./automation/) - Infrastructure Automation
**Perfect for**: Infrastructure provisioning, Terraform workflows, AWS resource management
**Constraints**: None - unlimited resources and duration
**Features**: Infrastructure as Code, state management, resource validation

| Example | Use Case | Key Features | Best For |
|---------|----------|-------------|----------|
| [Terraform S3](./automation/terraform-s3.md) | Infrastructure automation | Terraform state management, AWS integration, resource verification | Cloud infrastructure, S3 provisioning |

## Getting Started

### 1. Choose Your Build Type
- **Go applications**: Start with [Go Build](./build/go-build.md)
- **Containerized apps**: Use [Docker Build](./build/docker-build.md)
- **Java/Spring Boot**: Begin with [Java Build](./build/java-build.md)

### 2. Advanced Features to Explore
- **Parallel execution**: See container sets in Go Build example
- **Retries & timeouts**: Demonstrated in Docker Build example
- **Variables & secrets**: Comprehensive usage in Java Build example

## Implementation Guide

### Step 1: Copy Example Configuration
1. Choose the appropriate example for your technology stack
2. Copy the YAML content from the example file
3. Paste it into your pipeline configuration

### Step 2: Configure Variables and Secrets
Each example includes a configuration section at the bottom. Update these in your pipeline:

```yaml
# Example configuration (set in Choreo Console)
VARIABLES:
  APP_NAME: "my-application"
  VERSION: "1.0.0"
  BUILD_NUMBER: "123"

SECRETS:
  DATABASE_URL: "your-database-url"
  API_TOKEN: "your-secret-token"
```

### Step 3: Customize for Your Needs
- Update container images to match your requirements
- Modify timeout and retry settings based on your application
- Add or remove steps as needed
- Adjust resource allocations in container sets

## Quick Reference

### Key Features by Example
- **Go Build**: Container sets, parallel execution, cross-platform builds
- **Docker Build**: Multi-stage builds, security scanning, comprehensive retries
- **Java Build**: Complex variable management, database integration, code quality

### When to Use Each Pattern
- **Container Sets**: When you need specific resource allocations or custom images
- **Parallel Execution**: To speed up testing and validation steps
- **Retries & Timeouts**: For network-dependent operations and external service calls
- **Variables & Secrets**: For environment-specific configuration and sensitive data
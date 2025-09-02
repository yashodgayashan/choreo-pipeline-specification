# Choreo Templates API Reference

## Overview

Choreo provides a set of pre-configured templates for common pipeline tasks. These templates are maintained by the Choreo team and follow best practices for security, performance, and reliability.

**Note**: Choreo templates are currently optimized for **Build Pipelines** (`pipelineType: build`), which provide fast feedback for code changes with built-in constraints and automatic source checkout.

**Important**: Template arguments are not currently supported. Templates must be used without custom parameters.

This document provides an API reference overview. For detailed usage examples and comprehensive documentation, see the [Choreo Templates Guide](../choreo-templates/overview.md).

## Template Naming Convention

Choreo templates follow the naming pattern:

```
choreo/<template-name>@<version>
```

- **Namespace**: `choreo/` indicates a built-in template
- **Template Name**: Descriptive name of the template  
- **Version**: Semantic version (e.g., `@v1`, `@v2`)

## Template Categories

### Build Templates
Templates for building applications across different languages and frameworks. **Optimized for Build Pipelines (`pipelineType: ci`).**

| Template | Purpose | Documentation |
|----------|---------|---------------|
| `choreo/buildpack-build@v1` | Cloud Native Buildpacks builder | [Build Templates](../choreo-templates/build-templates.md#chorebuildpack-buildv1) |
| `choreo/docker-build@v1` | Docker image builder | [Build Templates](../choreo-templates/build-templates.md#choreodocker-buildv1) |

### Security Templates
Templates for comprehensive security scanning capabilities. **Optimized for Build Pipelines (`pipelineType: ci`).**

| Template | Purpose | Documentation |
|----------|---------|---------------|
| `choreo/trivy-scan@v1` | Container vulnerability scanning | [Security Templates](../choreo-templates/security-templates.md#choreotrivy-scanv1) |
| `choreo/checkov-scan@v1` | Infrastructure as Code security | [Security Templates](../choreo-templates/security-templates.md#choreocheckov-scanv1) |


## Usage Syntax

### Basic Template Usage (Build Pipeline)

```yaml
steps:
  - name: Build Application
    template: choreo/buildpack-build@v1

  - name: Security Scan
    template: choreo/trivy-scan@v1
```

### Template Chaining (Complete Build Pipeline)

```yaml
steps:
  - name: Build
    template: choreo/docker-build@v1

  - name: Security Scan
    template: choreo/trivy-scan@v1

  - name: Infrastructure Security Check
    template: choreo/checkov-scan@v1
```

## Best Practices

### 1. Version Pinning
Always specify template versions:
```yaml
# Good - pinned version
template: choreo/docker-build@v1

# Avoid - may break with updates  
template: choreo/docker-build@latest
```

### 2. Error Handling
Configure appropriate retry and timeout strategies:
```yaml
steps:
  - name: Publish
    template: choreo/artifact-upload@v1
    retryStrategy:
      limit: 2
      retryPolicy: OnFailure
    timeout: "10m"
```

## See Also

- **[Choreo Templates Overview](../choreo-templates/overview.md)** - Comprehensive templates guide
- **[Pipeline Configuration](../specification/pipeline-configuration.md)** - Complete pipeline syntax
- **[Migration Guides](../guides/migration.md)** - Platform-specific migration instructions
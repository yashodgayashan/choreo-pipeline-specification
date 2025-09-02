# Choreo Templates Reference

Choreo provides a comprehensive set of pre-built templates that simplify common CI tasks. These templates are maintained by the Choreo team and follow best practices for security, performance, and reliability.

**Current Focus**: Most Choreo templates are currently optimized for **Build Pipelines** (`pipelineType: ci`), which provide:

- Fast feedback on code changes
- Built-in security scanning
- Automatic source checkout and artifact management
- 60-minute execution limit with resource constraints

## Template Naming Convention

All Choreo templates follow the pattern:
```
choreo/<template-name>@<version>
```

## Available Templates

### Build Templates

#### `choreo/buildpack-build@v1`
Build applications using Cloud Native Buildpacks.

**Usage:**
```yaml
- name: Build Application
  template: choreo/buildpack-build@v1
```

#### `choreo/docker-build@v1`
Build Docker images with multi-platform support.

**Usage:**
```yaml
- name: Build Docker Image
  template: choreo/docker-build@v1
```

### Security Templates

#### `choreo/trivy-scan@v1`
Container and filesystem vulnerability scanning.

**Usage:**
```yaml
- name: Security Scan
  template: choreo/trivy-scan@v1
```

#### `choreo/checkov-scan@v1`
Infrastructure as Code security scanning.

**Usage:**
```yaml
- name: IaC Security Scan
  template: choreo/checkov-scan@v1
```

## Best Practices

### Version Pinning
Always specify template versions:
```yaml
# Good - pinned version
template: choreo/docker-build@v1

# Avoid - may break with updates
template: choreo/docker-build@latest
```

### Error Handling
Add appropriate error handling:
```yaml
steps:
  - name: Build
    template: choreo/docker-build@v1
    retryStrategy:
      limit: 2
      retryPolicy: OnFailure
```
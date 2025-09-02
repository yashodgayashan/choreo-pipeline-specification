# Troubleshooting Guide

## Overview

This guide helps diagnose and resolve common issues with Choreo pipelines.

## Common Issues

### Pipeline Not Running

#### Symptom
Pipeline doesn't trigger or appear in Choreo console.

#### Possible Causes & Solutions

**1. Invalid YAML Syntax**

- Validate with Choreo Console
- Validate with schema validation tool [Schema Validation Guide](../specification/schema-validation.md)


### Resource Errors

#### Symptom
"Insufficient resources" or "Resource quota exceeded" errors.

#### Solutions

**For CI Pipelines (Limited Resources)**
```yaml
# Exceeds CI limits
steps:
  - name: build
    resources:
      limits:
        memory: "16Gi"  # Too high for CI
        cpu: "8000m"     # Too high for CI

# Within CI limits
steps:
  - name: build
    resources:
      limits:
        memory: "4Gi"   # Max 8Gi for CI
        cpu: "2000m"    # Max 4000m for CI
```

**For Automation Pipelines (Flexible)**
```yaml
# Switch to automation type for unlimited resources
pipelineType: automation  # No resource limits
steps:
  - name: heavy-processing
    resources:
      limits:
        memory: "32Gi"  # OK for automation
        cpu: "16000m"   # OK for automation
```

### Variable Not Found

#### Symptom
"Variable not found" or "undefined variable" errors.

#### Solutions

**1. Check Variable Definition**
```yaml
# Ensure variable is defined in Choreo console
env:
  - name: API_URL
    value: "{{VARIABLES.API_URL}}"  # Must exist in console
```

**2. Correct Syntax**
```yaml
# Wrong syntax
value: "{{VAR.API_URL}}"        # Old syntax
value: "{{variables.api_url}}"  # Wrong case
value: "{{API_URL}}"            # Missing prefix

# Correct syntax
value: "{{VARIABLES.API_URL}}"  # Variables
value: "{{SECRETS.API_KEY}}"    # Secrets
```

**3. Check Scope**
```yaml
# Component-level variables
value: "{{VARIABLES.COMPONENT.DB_HOST}}"

# Global variables
value: "{{VARIABLES.DB_HOST}}"
```

### Step Failures

#### Symptom
Steps fail with non-zero exit codes.

#### Solutions

**1. Add Error Handling**
```yaml
steps:
  - name: flaky-step
    inlineScript: |
      #!/bin/bash
      set -e  # Exit on error
      
      # Add error handling
      if ! command -v node > /dev/null; then
        echo "Node.js not installed"
        exit 1
      fi
      
      npm test || {
        echo "Tests failed"
        exit 1
      }
```

**2. Add Retry Strategy**
```yaml
steps:
  - name: network-operation
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
      backoff:
        duration: "30s"
        factor: 2
```

**3. Continue on Non-Critical Failures**
```yaml
steps:
  - name: optional-step
    continueOn:
      error: true  # Don't fail pipeline
```

### Timeout Errors

#### Symptom
"Step timeout exceeded" or "Pipeline timeout exceeded".

#### Solutions

**1. Increase Step Timeout**
```yaml
steps:
  - name: long-running
    timeout: "30m"  # Increase from default
    template: process-data
```

**2. Check Pipeline Type Limits**
```yaml
# CI pipelines have 60-minute limit
pipelineType: ci
steps:
  - name: build
    timeout: "30m"  # Max 30m per step for CI

# Use automation for longer tasks
pipelineType: automation
steps:
  - name: process
    timeout: "4h"  # No limit for automation
```

### Permission Denied

#### Symptom
"Permission denied" errors when accessing files or resources.

#### Solutions

**1. Check Volume Permissions**
```yaml
steps:
  - name: write-data
    volumeMounts:
      - name: data
        mountPath: /data
        readOnly: false  # Ensure write access
```

**2. Run as Correct User**
```yaml
steps:
  - name: privileged-task
    securityContext:
      runAsUser: 0  # Run as root if needed
      runAsGroup: 0
```

### Docker Build Failures

#### Symptom
Docker build or push operations fail.

#### Solutions

**1. Docker-in-Docker Setup**
```yaml
steps:
  - name: docker-build
    containerSet:
      containers:
        - name: docker
          image: docker:dind
          securityContext:
            privileged: true
          command: ["dockerd-entrypoint.sh"]
        - name: build
          image: docker:latest
          env:
            - name: DOCKER_HOST
              value: tcp://localhost:2375
          command: ["sh", "-c", "docker build -t app ."]
```

**2. Registry Authentication**
```yaml
steps:
  - name: docker-push
    inlineScript: |
      #!/bin/bash
      echo "$DOCKER_PASSWORD" | docker login "$DOCKER_REGISTRY" \
        --username "$DOCKER_USERNAME" \
        --password-stdin
      
      docker push $IMAGE
    env:
      - name: DOCKER_REGISTRY
        value: "{{VARIABLES.DOCKER_REGISTRY}}"
      - name: DOCKER_USERNAME
        value: "{{VARIABLES.DOCKER_USERNAME}}"
      - name: DOCKER_PASSWORD
        value: "{{SECRETS.DOCKER_PASSWORD}}"
```

### Parallel Step Issues

#### Symptom
Parallel steps not executing concurrently.

#### Solutions

**Correct Parallel Syntax**
```yaml
# Wrong - sequential execution
steps:
  - name: test1
    template: test
  - name: test2
    template: test

# Correct - parallel execution
steps:
  - - name: test1  # Note double dash
      template: test
    - name: test2
      template: test
```

### Template Not Found

#### Symptom
"Template not found" errors.

#### Solutions

**1. Check Template Definition**
```yaml
steps:
  - name: build
    template: my-template  # Must be defined

templates:
  - name: my-template  # Define template
    inlineScript: |
      #!/bin/bash
      echo "Template logic"
```

**2. Built-in Template Version**
```yaml
# Wrong
template: choreo/docker-build  # Missing version

# Correct
template: choreo/docker-build@v1  # Include version
```

## Debugging Techniques

### Enable Debug Output

```yaml
steps:
  - name: debug-step
    inlineScript: |
      #!/bin/bash
      set -x  # Enable debug output
      
      echo "Debug: Current directory: $(pwd)"
      echo "Debug: Environment variables:"
      env | sort
      
      # Your actual commands
      npm test
```

### Check Environment

```yaml
steps:
  - name: check-environment
    inlineScript: |
      #!/bin/bash
      echo "=== System Information ==="
      echo "Hostname: $(hostname)"
      echo "OS: $(uname -a)"
      echo "Current User: $(whoami)"
      echo "Working Directory: $(pwd)"
      echo ""
      echo "=== Resource Information ==="
      echo "Memory:"
      free -h
      echo ""
      echo "CPU:"
      nproc
      echo ""
      echo "Disk:"
      df -h
      echo ""
      echo "=== Environment Variables ==="
      env | sort
```

### Validate Pipeline Configuration

```yaml
steps:
  - name: validate-config
    inlineScript: |
      #!/bin/bash
      echo "Pipeline Type: {{pipelineType}}"
      echo "Version: {{version}}"
      echo "Branch: $CI_BRANCH"
      echo "Pipeline ID: $CI_PIPELINE_ID"
      
      # Check if required tools are available
      for tool in git docker kubectl; do
        if command -v $tool > /dev/null; then
          echo "✓ $tool is available"
        else
          echo "✗ $tool is not available"
        fi
      done
```

## Error Messages Reference

### Common Error Messages and Solutions

| Error Message | Cause | Solution |
|--------------|-------|----------|
| `yaml: line X: found character that cannot start any token` | Invalid YAML syntax | Check indentation and special characters |
| `template not found: xxx` | Missing template definition | Define template or check spelling |
| `variable not found: xxx` | Undefined variable | Add variable in Choreo console |
| `resource quota exceeded` | Resource limits exceeded | Reduce resources or use automation pipeline |
| `timeout waiting for condition` | Step timeout | Increase timeout or optimize step |
| `permission denied` | Insufficient permissions | Check volume mounts and security context |
| `no space left on device` | Disk full | Clean up files or increase volume size |
| `image pull backoff` | Can't pull Docker image | Check image name and registry credentials |

## Performance Issues

### Slow Pipeline Execution

**1. Parallelize Steps**
```yaml
# Convert sequential to parallel
steps:
  - - name: test-unit
    - name: test-integration
    - name: lint
```

**2. Use Caching**
```yaml
volumes:
  - name: cache
    emptyDir: {}

steps:
  - name: build
    volumeMounts:
      - name: cache
        mountPath: /root/.cache
```

**3. Optimize Resource Allocation**
```yaml
steps:
  - name: build
    resources:
      requests:
        memory: "2Gi"  # Request what you need
        cpu: "2000m"
```

## Getting Support

### Before Contacting Support

1. **Check Logs**
   - Review step logs in Choreo console
   - Look for error messages and stack traces

2. **Validate Configuration**
   - Check YAML syntax
   - Verify resource limits
   - Confirm variable definitions

3. **Test Locally**
   - Run commands locally to isolate issues
   - Verify external dependencies

### Information to Provide

When contacting support, include:

1. Pipeline configuration (`Choreo Console configuration`)
2. Error messages and logs
3. Pipeline type and version
4. Steps to reproduce
5. Expected vs actual behavior

### Support Channels

- Documentation: Review guides and examples
- Community: Choreo community forums
- Support Tickets: For critical issues
- Slack: Team collaboration channel
# Pipeline Configuration

## Overview

This document provides a comprehensive guide to configuring pipelines using the Choreo Pipeline Specification. It covers all configuration options, from basic setups to advanced scenarios.

## File Location

Pipeline configuration must be placed at:
```
Choreo Console Pipeline Configuration
```

## Basic Structure

A minimal pipeline configuration:

```yaml
steps:
  - name: Build
    inlineScript: |
      #!/bin/bash
      echo "Building application..."
```

## Complete Configuration Structure

```yaml
# Volume claims (optional)
volumeClaimTemplates:
  - metadata:
      name: volume-name
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "1Gi"

# Pipeline steps (required)
steps:
  - name: step-name
    template: template-name
    # ... step configuration

# Template definitions (optional)
templates:
  - name: template-name
    inlineScript: |
      #!/bin/bash
      echo "Template logic"

# Container template definitions (optional)
containerTemplates:
  - name: container-template-name
    image: "image:tag"
    command: ["sh"]
```

## Steps Configuration

### Basic Step

```yaml
steps:
  - name: My Step
    inlineScript: |
      #!/bin/bash
      echo "Step execution"
```

### Step with Template

```yaml
steps:
  - name: Build Step
    template: build-template
```

### Step with Choreo Built-in Template

```yaml
steps:
  - name: Docker Build
    template: choreo/docker-build@v1
```

### Step with Container Set

```yaml
steps:
  - name: Multi-Container Step
    containerSet:
      containers:
        - name: app
          image: "app:latest"
          command: ["sh", "-c", "echo 'App running'"]
        - name: sidecar
          image: "sidecar:latest"
          command: ["sh", "-c", "echo 'Sidecar running'"]
```

### Step with Environment Variables

```yaml
steps:
  - name: Configured Step
    inlineScript: |
      #!/bin/bash
      echo "API URL: $API_URL"
      echo "Token: [HIDDEN]"
    env:
      - name: API_URL
        value: "{{VARIABLES.API_URL}}"
      - name: API_TOKEN
        value: "{{SECRETS.API_TOKEN}}"
```

### Step with Volume Mounts

```yaml
steps:
  - name: Step with Storage
    inlineScript: |
      #!/bin/bash
      echo "data" > /workspace/file.txt
    volumeMounts:
      - name: workspace-volume
        mountPath: /workspace
```

### Step with Resources

```yaml
steps:
  - name: Resource-Limited Step
    inlineScript: |
      #!/bin/bash
      echo "Running with resource limits"
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
```

### Step with Retry Strategy

```yaml
steps:
  - name: Retryable Step
    inlineScript: |
      #!/bin/bash
      # Potentially flaky operation
      curl https://api.example.com/health
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
      backoff:
        duration: "30s"
        factor: 2
        maxDuration: "5m"
```

### Parallel Steps

```yaml
steps:
  - name: Sequential Step 1
    template: step1
  - - name: Parallel Step A    # Note the double dash
    template: stepA
  - name: Parallel Step B
    template: stepB
  - name: Parallel Step C
    template: stepC
  - name: Sequential Step 2    # Executes after parallel steps
    template: step2
```

## Templates Configuration

### Choreo Template

```yaml
templates:
  - name: test-runner
    inlineScript: |
      #!/bin/bash
      npm install
      npm test
    image: "node:18-alpine"
    env:
      - name: NODE_ENV
        value: "test"
    volumeMounts:
      - name: cache-volume
        mountPath: /root/.npm
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
```

### Argo Script Template

```yaml
templates:
  - name: script-template
    script:
      image: "python:3.9"
      command: ["python"]
      source: |
        import json
        import sys
        
        result = {"status": "success"}
        with open("/tmp/result.json", "w") as f:
            json.dump(result, f)
      env:
        - name: PYTHONPATH
          value: "/app"
    outputs:
      parameters:
        - name: result
          valueFrom:
            path: /tmp/result.json
```

### Argo Container Set Template

```yaml
templates:
  - name: container-set-template
    containerSet:
      containers:
        - name: database
          image: "postgres:13"
          env:
            - name: POSTGRES_PASSWORD
              value: "{{SECRETS.DB_PASSWORD}}"
        - name: app
          image: "app:latest"
          dependencies: ["database"]
          env:
            - name: DB_HOST
              value: "localhost"
```

## Container Templates

```yaml
containerTemplates:
  - name: security-scanner
    image: "aquasec/trivy:latest"
    command: ["trivy"]
    source: |
      image --severity HIGH,CRITICAL $IMAGE_NAME
    env:
      - name: TRIVY_NO_PROGRESS
        value: "true"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
```

## Input/Output Configuration

### Template with Inputs

```yaml
templates:
  - name: parameterized-template
    inputs:
      parameters:
        - name: version
        - name: environment
    inlineScript: |
      #!/bin/bash
      echo "Deploying version {{inputs.parameters.version}}"
      echo "Target environment: {{inputs.parameters.environment}}"
```

### Template with Outputs

```yaml
templates:
  - name: output-generator
    inlineScript: |
      #!/bin/bash
      BUILD_ID=$(date +%s)
      echo "$BUILD_ID" > /tmp/build-id.txt
      echo '{"status":"success"}' > /tmp/result.json
    outputs:
      parameters:
        - name: build-id
          valueFrom:
            path: /tmp/build-id.txt
        - name: result
          valueFrom:
            path: /tmp/result.json
```

### Passing Parameters Between Steps

```yaml
steps:
  - name: generate
    template: output-generator
  - name: consume
    template: parameterized-template
    arguments:
      parameters:
        - name: version
          value: "{{steps.generate.outputs.parameters.build-id}}"
        - name: environment
          value: "production"
```

## Environment Variables

### Variable Types

```yaml
env:
  # Component variables
  - name: API_URL
    value: "{{VARIABLES.API_URL}}"

  # Component secrets
  - name: API_KEY
    value: "{{SECRETS.API_KEY}}"

  # Organization level variables and secrets
  - name: ORG_REGISTRY_URL
    value: "{{VARIABLES.ORG.REGISTRY_URL}}"
  - name: ORG_API_KEY
    value: "{{SECRETS.ORG.API_KEY}}"

  # Project level variables and secrets
  - name: PROJECT_DATABASE_URL
    value: "{{VARIABLES.PROJECT.DATABASE_URL}}"
  - name: PROJECT_DB_PASSWORD
    value: "{{SECRETS.PROJECT.DB_PASSWORD}}"

  # Component level variables and secrets
  - name: COMPONENT_PORT
    value: "{{VARIABLES.COMPONENT.PORT}}"
  - name: COMPONENT_SECRET_KEY
    value: "{{SECRETS.COMPONENT.SECRET_KEY}}"

  # Data Plane (DT) level variables and secrets
  - name: DT_ENDPOINT
    value: "{{VARIABLES.DT.ENDPOINT}}"
  - name: DT_ACCESS_TOKEN
    value: "{{SECRETS.DT.ACCESS_TOKEN}}"

  # Static values
  - name: LOG_LEVEL
    value: "info"
  
```

## Volume Configuration

### Volume Claim Template

```yaml
volumeClaimTemplates:
  - metadata:
      name: build-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "choreo-workflows-storage"
      resources:
        requests:
          storage: "10Gi"
```


## Resource Specifications

### Memory and CPU

```yaml
resources:
  requests:
    memory: "256Mi"    # Minimum memory
    cpu: "250m"        # 0.25 CPU cores
  limits:
    memory: "512Mi"    # Maximum memory
    cpu: "500m"        # 0.5 CPU cores
```

### Resource Units

- **Memory**: Use standard Kubernetes units (Ki, Mi, Gi)
- **CPU**: Use millicores (m) or cores (1000m = 1 core)

## Retry Strategies

### Retry Policies

- `Always`: Retry on both failure and error
- `OnFailure`: Retry only on failure
- `OnError`: Retry only on error

### Backoff Configuration

```yaml
retryStrategy:
  limit: 5                    # Maximum retry attempts
  retryPolicy: OnFailure      # When to retry
  backoff:
    duration: "10s"           # Initial backoff duration
    factor: 2                 # Backoff multiplier
    maxDuration: "5m"         # Maximum backoff duration
```

## Security Considerations

1. **Never hardcode secrets** - Always use `{{SECRETS.NAME}}`
2. **Use minimal base images** - Reduce attack surface
3. **Set resource limits** - Prevent resource exhaustion
4. **Run as non-root** - Use security context when possible
5. **Scan images** - Use security scanning templates

## Validation

Before deploying, validate your pipeline:

1. YAML syntax is correct
2. All referenced templates exist
3. Required environment variables are defined
4. Resource limits are reasonable
5. Secrets are properly referenced

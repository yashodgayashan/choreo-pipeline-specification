# Core Concepts

## Overview

This document explains the fundamental concepts of the Choreo Pipeline Specification. Understanding these concepts is essential for creating effective and maintainable pipelines.

## Pipeline Structure

A pipeline consists of a series of steps that execute either sequentially or in parallel. Each pipeline is configured through the Choreo Console interface, where you can input the YAML configuration directly or point to a repository file.

## Steps

Steps are the basic building blocks of a pipeline. Each step represents a single task or operation.

### Step Types

1. **Template-based Steps**: Reference pre-defined templates
2. **Inline Script Steps**: Define scripts directly in the step
3. **Container Set Steps**: Run multiple containers together

### Step Execution

- **Sequential**: Steps execute one after another (default)
- **Parallel**: Steps execute simultaneously (using `- -` syntax)

```yaml
steps:
  - name: Setup           # Executes first
    template: setup
  - - name: Test A        # These three steps
    - name: Test B        # execute in
    - name: Test C        # parallel
  - name: Cleanup         # Executes after parallel steps complete
```

## Templates

Templates are reusable units of work that encapsulate logic, configuration, and dependencies.

### Template Types

1. **Choreo Templates**: Simplified template format
2. **Argo Templates**: Advanced template format (script and containerSet only)
3. **Built-in Templates**: Pre-configured templates provided by Choreo

### Choreo Templates

Choreo templates provide a simplified way to define reusable components:

```yaml
templates:
  - name: my-template
    inlineScript: |
      #!/bin/bash
      echo "Hello from template"
    image: "alpine:3.18"
    env:
      - name: MY_VAR
        value: "value"
```

### Argo Templates

For advanced use cases, Argo templates offer more flexibility:

```yaml
templates:
  - name: advanced-template
    script:
      image: "node:18"
      command: ["sh"]
      source: |
        #!/bin/bash
        echo "Advanced template"
```

## Container Templates

Container templates define reusable container configurations:

```yaml
containerTemplates:
  - name: node-runner
    image: "node:18-alpine"
    command: ["sh"]
    source: |
      #!/bin/bash
      npm install
      npm test
```

## Environment Variables

Three types of environment variables are supported:

### 1. Variables (Configuration)

Non-sensitive configuration values managed through Choreo console:

```yaml
env:
  - name: API_ENDPOINT
    value: "{{VARIABLES.API_ENDPOINT}}"
```

### 2. Secrets (Sensitive Data)

Sensitive values securely managed through Choreo console:

```yaml
env:
  - name: API_TOKEN
    value: "{{SECRETS.API_TOKEN}}"
```

### 3. Hierarchical Variables and Secrets

CI pipelines have access to variables and secrets at different organizational levels:

#### Organization Level
```yaml
env:
  - name: ORG_REGISTRY_URL
    value: "{{VARIABLES.ORG.REGISTRY_URL}}"
  - name: ORG_API_KEY
    value: "{{SECRETS.ORG.API_KEY}}"
```

#### Project Level
```yaml
env:
  - name: PROJECT_DATABASE_URL
    value: "{{VARIABLES.PROJECT.DATABASE_URL}}"
  - name: PROJECT_DB_PASSWORD
    value: "{{SECRETS.PROJECT.DB_PASSWORD}}"
```

#### Component Level
```yaml
env:
  - name: COMPONENT_PORT
    value: "{{VARIABLES.COMPONENT.PORT}}"
  - name: COMPONENT_SECRET_KEY
    value: "{{SECRETS.COMPONENT.SECRET_KEY}}"
```

#### Data Plane (DT) Level
```yaml
env:
  - name: DT_ENDPOINT
    value: "{{VARIABLES.DT.ENDPOINT}}"
  - name: DT_ACCESS_TOKEN
    value: "{{SECRETS.DT.ACCESS_TOKEN}}"
```

### 3. Inline Values

Static values defined directly in the pipeline:

```yaml
env:
  - name: NODE_ENV
    value: "production"
```

## Volume Management

### Volume Claims

Persistent storage that survives across steps:

```yaml
volumeClaimTemplates:
  - metadata:
      name: workspace-volume
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "1Gi"
```

### Volume Mounts

Attach volumes to steps:

```yaml
volumeMounts:
  - name: workspace-volume
    mountPath: /workspace
```

## Input/Output Parameters

Pass data between steps using parameters:

### Outputs

Define what a step produces:

```yaml
outputs:
  parameters:
    - name: build-id
      valueFrom:
        path: /tmp/build-id.txt
```

### Inputs

Define what a step requires:

```yaml
inputs:
  parameters:
    - name: build-id
```

### Parameter Passing

Reference outputs in subsequent steps:

```yaml
steps:
  - name: generate
    template: generate-id
  - name: consume
    template: use-id
    arguments:
      parameters:
        - name: id
          value: "{{steps.generate.outputs.parameters.build-id}}"
```

## Resource Management

Control CPU and memory allocation:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1000m"
```

## Retry Strategies

Configure automatic retry behavior:

```yaml
retryStrategy:
  limit: 3
  retryPolicy: OnFailure
  backoff:
    duration: "30s"
    factor: 2
    maxDuration: "5m"
```

## Best Practices

1. **Use Templates**: Create reusable templates for common tasks
2. **Manage Secrets Properly**: Never hardcode sensitive data
3. **Set Resource Limits**: Prevent resource exhaustion
4. **Add Retry Logic**: Handle transient failures gracefully
5. **Use Parallel Execution**: Optimize pipeline duration
6. **Pass Parameters**: Share data between steps efficiently
7. **Document Templates**: Add comments to explain complex logic

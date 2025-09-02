# Templates API Reference

## Overview

Templates are reusable units of work that encapsulate pipeline logic, configuration, and dependencies. They promote code reuse and maintainability across pipelines.

## Template Types

### 1. Choreo Templates
Simplified template format for common use cases

### 2. Argo Templates  
Advanced template format with more control (limited to script and containerSet)

### 3. Built-in Templates
Pre-configured templates provided by Choreo (e.g., `choreo/docker-build@v1`)

## Choreo Template Definition

```yaml
templates:
  - name: <string>              # Required: Unique template name
    inlineScript: <string>      # Required: Bash script to execute
    image: <string>             # Optional: Container image
    env: <array>                # Optional: Environment variables
    volumeMounts: <array>       # Optional: Volume mounts
    resources: <object>         # Optional: Resource limits
    inputs: <object>            # Optional: Input parameters
    outputs: <object>           # Optional: Output parameters
```

## Argo Template Definition

### Script Template

```yaml
templates:
  - name: <string>              # Required: Template name
    script:                     # Required: Script configuration
      image: <string>           # Required: Container image
      command: <array>          # Optional: Command to execute
      source: <string>          # Required: Script source code
      env: <array>              # Optional: Environment variables
      volumeMounts: <array>     # Optional: Volume mounts
      resources: <object>       # Optional: Resource limits
    inputs: <object>            # Optional: Input parameters
    outputs: <object>           # Optional: Output parameters
```

### Container Set Template

```yaml
templates:
  - name: <string>              # Required: Template name
    containerSet:               # Required: Container set config
      containers: <array>       # Required: Container definitions
    inputs: <object>            # Optional: Input parameters
    outputs: <object>           # Optional: Output parameters
```

## Fields

### `name` (Required)

**Type:** `string`  
**Description:** Unique identifier for the template. Must be alphanumeric with optional hyphens or underscores.

```yaml
templates:
  - name: build-template
```

### `inlineScript` (Choreo Templates)

**Type:** `string`  
**Description:** Bash script to execute. Must start with shebang (`#!/bin/bash`).

```yaml
templates:
  - name: test-runner
    inlineScript: |
      #!/bin/bash
      echo "Running tests..."
      npm test
```

### `image` (Optional)

**Type:** `string`  
**Description:** Docker image to use for execution. Defaults to Choreo base image if not specified.

```yaml
templates:
  - name: node-build
    image: "node:18-alpine"
    inlineScript: |
      #!/bin/bash
      npm install
      npm run build
```

### `script` (Argo Templates)

**Type:** `object`  
**Description:** Script configuration for Argo script templates.

#### Script Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `image` | string | Yes | Container image |
| `command` | array | No | Command to execute |
| `source` | string | Yes | Script source code |
| `env` | array | No | Environment variables |
| `volumeMounts` | array | No | Volume mounts |
| `resources` | object | No | Resource limits |

```yaml
templates:
  - name: python-script
    script:
      image: "python:3.9"
      command: ["python"]
      source: |
        import json
        import sys
        
        result = {"status": "success"}
        print(json.dumps(result))
```

### `containerSet` (Argo Templates)

**Type:** `object`  
**Description:** Configuration for running multiple containers.

#### Container Set Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `containers` | array | Yes | Container definitions |

#### Container Definition

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Container name |
| `image` | string | Yes | Container image |
| `command` | array | No | Command to execute |
| `args` | array | No | Command arguments |
| `env` | array | No | Environment variables |
| `resources` | object | No | Resource limits |
| `dependencies` | array | No | Container dependencies |

```yaml
templates:
  - name: multi-container
    containerSet:
      containers:
        - name: database
          image: postgres:13
          env:
            - name: POSTGRES_PASSWORD
              value: "{{SECRETS.DB_PASSWORD}}"
        - name: app
          image: myapp:latest
          dependencies: ["database"]
```

### `env` (Optional)

**Type:** `array`  
**Description:** Environment variables for the template.

```yaml
templates:
  - name: configured-template
    env:
      - name: NODE_ENV
        value: "production"
      - name: API_KEY
        value: "{{SECRETS.API_KEY}}"
    inlineScript: |
      #!/bin/bash
      echo "Environment: $NODE_ENV"
```

### `volumeMounts` (Optional)

**Type:** `array`  
**Description:** Volume mount configurations.

```yaml
templates:
  - name: cache-template
    volumeMounts:
      - name: npm-cache
        mountPath: /root/.npm
      - name: workspace
        mountPath: /workspace
    inlineScript: |
      #!/bin/bash
      npm install --cache /root/.npm
```

### `resources` (Optional)

**Type:** `object`  
**Description:** CPU and memory resource specifications.

```yaml
templates:
  - name: resource-limited
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      npm run build
```

### `inputs` (Optional)

**Type:** `object`  
**Description:** Input parameter definitions for the template.

#### Input Structure

```yaml
inputs:
  parameters:
    - name: <string>            # Parameter name
      value: <string>           # Optional: Default value
```

Example:

```yaml
templates:
  - name: parameterized
    inputs:
      parameters:
        - name: environment
        - name: version
          value: "latest"
    inlineScript: |
      #!/bin/bash
      echo "Deploying version {{inputs.parameters.version}}"
      echo "Target: {{inputs.parameters.environment}}"
```

### `outputs` (Optional)

**Type:** `object`  
**Description:** Output parameter definitions for the template.

#### Output Structure

```yaml
outputs:
  parameters:
    - name: <string>            # Parameter name
      valueFrom:
        path: <string>          # File path to read value from
```

Example:

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

## Template Reference

### Using Templates in Steps

```yaml
steps:
  - name: build
    template: my-build-template
    arguments:
      parameters:
        - name: target
          value: "production"
```

### Template Inheritance

Templates can reference other templates:

```yaml
templates:
  - name: base-template
    inlineScript: |
      #!/bin/bash
      echo "Base setup"
  
  - name: extended-template
    template: base-template
    env:
      - name: EXTENDED
        value: "true"
```

## Examples

### Basic Choreo Template

```yaml
templates:
  - name: simple-test
    inlineScript: |
      #!/bin/bash
      echo "Running tests..."
      npm test
      echo "Tests completed"
```

### Parameterized Template

```yaml
templates:
  - name: deploy
    inputs:
      parameters:
        - name: environment
        - name: replicas
          value: "3"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to {{inputs.parameters.environment}}"
      kubectl scale deployment/app --replicas={{inputs.parameters.replicas}}
```

### Template with Outputs

```yaml
templates:
  - name: build-info
    inlineScript: |
      #!/bin/bash
      VERSION=$(git describe --tags --always)
      TIMESTAMP=$(date +%Y%m%d%H%M%S)
      echo "$VERSION" > /tmp/version.txt
      echo "$TIMESTAMP" > /tmp/timestamp.txt
    outputs:
      parameters:
        - name: version
          valueFrom:
            path: /tmp/version.txt
        - name: timestamp
          valueFrom:
            path: /tmp/timestamp.txt
```

### Argo Script Template

```yaml
templates:
  - name: python-processor
    script:
      image: python:3.9-slim
      command: ["python"]
      source: |
        import json
        import os
        
        # Process environment variables
        env_data = {
          "processed": True,
          "environment": os.environ.get("ENVIRONMENT", "unknown")
        }
        
        # Write output
        with open("/tmp/output.json", "w") as f:
          json.dump(env_data, f)
      env:
        - name: ENVIRONMENT
          value: "{{VARIABLES.ENVIRONMENT}}"
    outputs:
      parameters:
        - name: result
          valueFrom:
            path: /tmp/output.json
```

### Container Set Template

```yaml
templates:
  - name: integration-test
    containerSet:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_DB
              value: testdb
            - name: POSTGRES_PASSWORD
              value: "{{SECRETS.TEST_DB_PASSWORD}}"
        
        - name: redis
          image: redis:6-alpine
        
        - name: test-runner
          image: test-runner:latest
          dependencies: ["postgres", "redis"]
          command: ["sh", "-c"]
          args: ["sleep 10 && npm run test:integration"]
          env:
            - name: DB_HOST
              value: localhost
            - name: REDIS_HOST
              value: localhost
```

## Best Practices

1. **Use descriptive names** - Template names should clearly indicate their purpose
2. **Parameterize templates** - Make templates reusable with input parameters
3. **Document outputs** - Clearly document what outputs a template produces
4. **Set resource limits** - Always specify resource constraints
5. **Handle errors** - Include error handling in scripts
6. **Version templates** - Use semantic versioning for template changes
7. **Keep templates focused** - Each template should have a single responsibility
8. **Use environment variables** - Avoid hardcoding values in templates
9. **Test templates** - Validate templates work in isolation
10. **Share common templates** - Create a library of reusable templates
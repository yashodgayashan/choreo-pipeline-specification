# Steps API Reference

## Overview

Steps are the fundamental building blocks of a pipeline. Each step represents a single task or operation that executes as part of the pipeline.

## Step Definition

```yaml
steps:
  - name: <string>              # Required: Unique alphanumeric name
    template: <string>          # Template reference
    inlineScript: <string>      # Inline bash script
    containerSet: <object>      # Container set configuration
    env: <array>                # Environment variables
    volumeMounts: <array>       # Volume mounts
    resources: <object>         # Resource limits
    retryStrategy: <object>     # Retry configuration
    timeout: <string>           # Execution timeout
    when: <string>              # Conditional execution
    arguments: <object>         # Step arguments
    continueOn: <object>        # Error handling
```

## Fields

### `name` (Required)

**Type:** `string`  
**Description:** Unique name for the step. Must be alphanumeric with optional hyphens or underscores.

```yaml
steps:
  - name: build-application
```

### `template` (Conditional)

**Type:** `string`  
**Description:** Reference to a template. Can be a custom template or built-in Choreo template.

```yaml
steps:
  - name: build
    template: my-template         # Custom template
  - name: scan
    template: choreo/trivy-scan@v1  # Built-in template
```

### `inlineScript` (Conditional)

**Type:** `string`  
**Description:** Inline bash script to execute. Must start with shebang.

```yaml
steps:
  - name: test
    inlineScript: |
      #!/bin/bash
      echo "Running tests..."
      npm test
```

### `containerSet` (Conditional)

**Type:** `object`  
**Description:** Configuration for running multiple containers.

```yaml
steps:
  - name: integration-test
    containerSet:
      containers:
        - name: database
          image: postgres:13
        - name: app
          image: myapp:latest
```

### `env` (Optional)

**Type:** `array`  
**Description:** Environment variables for the step.

```yaml
steps:
  - name: deploy
    env:
      - name: ENVIRONMENT
        value: "production"
      - name: API_KEY
        value: "{{SECRETS.API_KEY}}"
```

### `volumeMounts` (Optional)

**Type:** `array`  
**Description:** Volume mount configurations.

```yaml
steps:
  - name: cache-build
    volumeMounts:
      - name: build-cache
        mountPath: /cache
      - name: workspace
        mountPath: /workspace
```

### `resources` (Optional)

**Type:** `object`  
**Description:** CPU and memory resource specifications.

```yaml
steps:
  - name: intensive-task
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
```

### `retryStrategy` (Optional)

**Type:** `object`  
**Description:** Retry behavior configuration.

```yaml
steps:
  - name: flaky-operation
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
      backoff:
        duration: "30s"
        factor: 2
        maxDuration: "5m"
```

#### Retry Strategy Fields

| Field | Type | Description |
|-------|------|-------------|
| `limit` | integer | Maximum retry attempts |
| `retryPolicy` | string | When to retry: `Always`, `OnFailure`, `OnError` |
| `backoff` | object | Backoff configuration |
| `backoff.duration` | string | Initial backoff duration |
| `backoff.factor` | number | Backoff multiplier |
| `backoff.maxDuration` | string | Maximum backoff duration |

### `timeout` (Optional)

**Type:** `string`  
**Description:** Maximum execution time for the step.

```yaml
steps:
  - name: long-running
    timeout: "30m"
```

### `when` (Optional)

**Type:** `string`  
**Description:** Conditional execution expression.

```yaml
steps:
  - name: production-only
    when: "{{arguments.parameters.environment}} == 'production'"
```

### `arguments` (Optional)

**Type:** `object`  
**Description:** Arguments to pass to the template.

```yaml
steps:
  - name: parameterized
    template: my-template
    arguments:
      parameters:
        - name: version
          value: "1.2.3"
        - name: config
          yamlObject:
            debug: true
            level: "info"
```

### `continueOn` (Optional)

**Type:** `object`  
**Description:** Error handling behavior.

```yaml
steps:
  - name: optional-step
    continueOn:
      error: true    # Continue on error
      failed: true   # Continue on failure
```

## Parallel Execution

Use double dash (`- -`) to execute steps in parallel:

```yaml
steps:
  - name: setup
    template: setup
  
  # These three steps run in parallel
  - - name: test-unit
      template: unit-tests
    - name: test-integration
      template: integration-tests  
    - name: lint
      template: lint-check
  
  - name: cleanup
    template: cleanup
```

## Step Execution Order

1. Steps execute sequentially by default
2. Parallel steps execute simultaneously
3. Next sequential step waits for all parallel steps
4. Steps with `when` conditions are evaluated before execution
5. Failed steps stop pipeline unless `continueOn` is set

## Examples

### Basic Step

```yaml
steps:
  - name: hello
    inlineScript: |
      #!/bin/bash
      echo "Hello, World!"
```

### Step with Template

```yaml
steps:
  - name: build
    template: build-template
    arguments:
      parameters:
        - name: target
          value: "production"
```

### Step with Resources and Retry

```yaml
steps:
  - name: test
    inlineScript: |
      #!/bin/bash
      npm test
    resources:
      limits:
        memory: "2Gi"
        cpu: "2"
    retryStrategy:
      limit: 2
      retryPolicy: OnFailure
```

### Conditional Step

```yaml
steps:
  - name: deploy-prod
    template: deploy
    when: "{{workflow.outputs.parameters.tests}} == 'passed'"
    arguments:
      parameters:
        - name: environment
          value: "production"
```

### Step with Container Set

```yaml
steps:
  - name: integration-test
    containerSet:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_PASSWORD
              value: "test"
        - name: redis
          image: redis:6
        - name: test-runner
          image: test-runner:latest
          dependencies: ["postgres", "redis"]
```

## Best Practices

1. **Use descriptive names** - Make step purpose clear
2. **Set resource limits** - Prevent resource exhaustion
3. **Add retry strategies** - Handle transient failures
4. **Use templates** - Avoid duplicating logic
5. **Set timeouts** - Prevent hanging pipelines
6. **Handle errors gracefully** - Use `continueOn` for non-critical steps
7. **Minimize step dependencies** - Use parallel execution when possible
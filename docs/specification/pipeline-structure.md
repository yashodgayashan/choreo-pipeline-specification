# Pipeline Structure API Reference

## Overview

This document defines the complete structure of a Choreo pipeline configuration file. A pipeline is configured through the Choreo Console either by direct YAML input or by referencing a YAML file in your repository. The specification supports multiple pipeline types including Build pipelines and general-purpose automation workflows.

## Top-Level Structure

```yaml

# Global arguments (optional)
arguments: <object>

# Volume claim templates (optional)
volumeClaimTemplates: <array>

# Pipeline steps (required)
steps: <array>

# Template definitions (optional)
templates: <array>

# Container template definitions (optional)
containerTemplates: <array>
```

## Field Descriptions

### `arguments` (Optional)

**Type:** `object`  
**Description:** Global parameters available to all steps and templates

```yaml
arguments:
  parameters:
    - name: environment
      value: "production"
    - name: version
      value: "{{VARIABLES.APP_VERSION}}"
```

#### Arguments Structure

```yaml
arguments:
  parameters:
    - name: <string>            # Parameter name
      value: <string>           # Parameter value
      yamlObject: <object>      # Alternative: YAML object value
```

### `volumeClaimTemplates` (Optional)

**Type:** `array`  
**Description:** Persistent volume claim templates for durable storage

```yaml
volumeClaimTemplates:
  - metadata:
      name: workspace-pvc
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "10Gi"
```

### `steps` (Required)

**Type:** `array`  
**Description:** Pipeline steps to execute sequentially or in parallel

```yaml
steps:
  - name: build
    template: build-template
  - - name: Unit Tests
      template: unit-tests     
    - name: Integration Tests
      template: integration-tests
  - name: deploy
    template: deploy-template
```

### `templates` (Optional)

**Type:** `array`  
**Description:** Reusable template definitions

```yaml
templates:
  - name: build-template
    inlineScript: |
      #!/bin/bash
      npm run build
```

### `containerTemplates` (Optional)

**Type:** `array`  
**Description:** Reusable container template definitions

```yaml
containerTemplates:
  - name: node-container
    image: "node:18"
    command: ["npm", "test"]
```

## Complete Pipeline Example

```yaml
# Persistent storage
volumeClaimTemplates:
  - metadata:
      name: artifact-storage
      labels:
        purpose: artifacts
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "choreo-workflows-storage"
      resources:
        requests:
          storage: "10Gi"

# Pipeline execution steps
steps:
  # Setup phase
  - name: checkout
    template: git-checkout
    arguments:
      parameters:
        - name: branch
          value: "{{VARIABLES.BRANCH}}"

  # Parallel quality checks
  - - name: lint
      template: code-lint
    - name: security-scan
      template: security-check
    - name: license-check
      template: license-validator

  # Build phase
  - name: build
    template: application-build
    arguments:
      parameters:
        - name: build_mode
          value: "production"
    volumeMounts:
      - name: build-output
        mountPath: /dist

  # Test phase
  - name: test
    template: test-runner
    arguments:
      parameters:
        - name: coverage_threshold
          value: "80"
    volumeMounts:
      - name: build-output
        mountPath: /dist
        readOnly: true

  # Package phase
  - name: package
    template: docker-package
    arguments:
      parameters:
        - name: image_name
          value: "{{arguments.parameters.registry}}/myapp"
        - name: image_tag
          value: "{{VARIABLES.VERSION}}"

  # Deploy phase
  - name: deploy
    template: choreo/deploy@v1
    when: "{{arguments.parameters.environment}} == 'production'"
    arguments:
      parameters:
        - name: environment
          value: "{{arguments.parameters.environment}}"

# Template definitions
templates:
  - name: git-checkout
    inputs:
      parameters:
        - name: branch
    inlineScript: |
      #!/bin/bash
      git checkout {{inputs.parameters.branch}}
      git pull origin {{inputs.parameters.branch}}

  - name: code-lint
    image: "node:{{arguments.parameters.node_version}}-alpine"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm install --frozen-lockfile
      npm run lint
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"

  - name: security-check
    template: choreo/trivy-scan@v1

  - name: license-validator
    inlineScript: |
      #!/bin/bash
      license-checker --production --summary

  - name: application-build
    inputs:
      parameters:
        - name: build_mode
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm install --frozen-lockfile
      npm run build:{{inputs.parameters.build_mode}}
      cp -r dist/* /dist/
    outputs:
      parameters:
        - name: build_id
          valueFrom:
            path: /dist/build-id.txt
    env:
      - name: NODE_ENV
        value: "{{inputs.parameters.build_mode}}"
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "2Gi"
        cpu: "2000m"

  - name: test-runner
    inputs:
      parameters:
        - name: coverage_threshold
    script:
      image: "node:{{arguments.parameters.node_version}}"
      command: ["sh"]
      source: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        npm test -- --coverage
        
        # Check coverage
        COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
        THRESHOLD={{inputs.parameters.coverage_threshold}}
        
        if (( $(echo "$COVERAGE < $THRESHOLD" | bc -l) )); then
          echo "Coverage too low: $COVERAGE%"
          exit 1
        fi
    outputs:
      parameters:
        - name: coverage
          valueFrom:
            path: /tmp/coverage.txt

  - name: docker-package
    inputs:
      parameters:
        - name: image_name
        - name: image_tag
    containerSet:
      containers:
        - name: docker-build
          image: docker:dind
          command: ["sh", "-c"]
          args:
            - |
              docker build -t {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}} .
              docker push {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}
          env:
            - name: DOCKER_HOST
              value: tcp://localhost:2375
          volumeMounts:
            - name: build-output
              mountPath: /dist
        - name: docker-daemon
          image: docker:dind
          command: ["dockerd-entrypoint.sh"]
          securityContext:
            privileged: true

# Container template definitions
containerTemplates:
  - name: base-node
    image: "node:{{arguments.parameters.node_version}}-alpine"
    command: ["sh"]
    env:
      - name: NODE_ENV
        value: "production"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
```

## Execution Flow

### Sequential Execution

Steps execute in order by default:

```yaml
steps:
  - name: step1    # Executes first
  - name: step2    # Executes after step1
  - name: step3    # Executes after step2
```

### Parallel Execution

Use double-dash syntax for parallel steps:

```yaml
steps:
  - name: setup           # Executes first
  - - name: parallel1     # These three execute
    - name: parallel2     # in parallel after
    - name: parallel3     # setup completes
  - name: cleanup         # Executes after all parallel steps
```

### Conditional Execution

Use `when` clause for conditional steps:

```yaml
steps:
  - name: test
    template: run-tests
  - name: deploy
    template: deploy-app
    when: "{{steps.test.outputs.parameters.status}} == 'passed'"
```

## Variable Scope

### Scope Hierarchy

1. **Step-level** (highest priority)
2. **Template-level**
3. **Global arguments**
4. **System variables** (lowest priority)

### Variable Resolution

```yaml
arguments:
  parameters:
    - name: version
      value: "1.0"        # Global scope

templates:
  - name: build
    env:
      - name: VERSION
        value: "2.0"      # Template scope

steps:
  - name: build-step
    template: build
    env:
      - name: VERSION
        value: "3.0"      # Step scope (wins)
```

## Validation Rules

### Required Fields

- At least one step must be defined
- Each step must have a unique name
- Each template must have a unique name

### Naming Conventions

- Names must be alphanumeric with hyphens/underscores
- Names must start with a letter
- Names are case-sensitive

### Reference Validation

- Template references must exist
- Volume references must match defined volumes
- Parameter references must be valid

## Best Practices

### 1. Structure Organization

```yaml
# Order sections logically
version: "1.0"
arguments: {}           # Configuration
steps: []              # Execution
templates: []          # Definitions
```

### 2. Use Comments

```yaml
# Build and test pipeline for Node.js application
steps:
  # Parallel quality checks
  - - name: lint         # Code style
    - name: test        # Unit tests
    - name: security    # Vulnerability scan
```

### 3. Modular Templates

```yaml
templates:
  # Small, focused templates
  - name: npm-install
    inlineScript: |
      npm install --frozen-lockfile
  
  - name: npm-build
    inlineScript: |
      npm run build
```

### 4. Parameter Defaults

```yaml
arguments:
  parameters:
    - name: timeout
      value: "5m"        # Sensible default
```

## Migration Guide

### From Jenkins

```groovy
// Jenkins
pipeline {
  stages {
    stage('Build') {
      steps {
        sh 'npm build'
      }
    }
  }
}
```

```yaml
# Choreo
steps:
  - name: Build
    inlineScript: |
      #!/bin/bash
      npm build
```

### From GitHub Actions

```yaml
# GitHub Actions
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm build
```

```yaml
# Choreo
steps:
  - name: checkout
    template: choreo/git-clone@v1
  - name: build
    inlineScript: |
      #!/bin/bash
      npm build
```

### From GitLab Pipeline

```yaml
# GitLab CI
build:
  stage: build
  script:
    - npm install
    - npm build
```

```yaml
# Choreo
steps:
  - name: build
    inlineScript: |
      #!/bin/bash
      npm install
      npm build
```

## Limitations

1. **Template Types**: Only script and containerSet templates supported
2. **Parallelism**: Single level of parallel execution
3. **DAG**: No support for complex DAG workflows
4. **Loops**: No native loop constructs
5. **Dynamic Steps**: Steps cannot be generated dynamically

## Error Handling

### Step Failures

By default, pipeline stops on first failure:

```yaml
steps:
  - name: may-fail
    template: flaky-test
    continueOn:
      error: true        # Continue even if this fails
```

### Retry Logic

```yaml
steps:
  - name: retryable
    template: network-call
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
```

## Debugging

### Debug Output

```yaml
steps:
  - name: debug
    inlineScript: |
      #!/bin/bash
      set -x  # Enable debug output
      echo "Debug: Variable = $MY_VAR"
      env | sort  # Show all environment variables
```

### Validation

Validate pipeline syntax:

```bash
# Validate against schema
# Validation is handled automatically by Choreo Console
```
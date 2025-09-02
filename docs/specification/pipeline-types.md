# Pipeline Types Specification

## Overview

The Choreo Pipeline Specification supports different pipeline types, each with specific characteristics, constraints, and capabilities. This document defines the differences between Build pipelines and Automation pipelines.


## Build Pipelines

### Purpose
Build  pipelines are specifically designed for building, testing, and validating code changes. They run in response to code commits and pull requests.

### Constraints

Build pipelines have the following constraints to ensure fast feedback and deterministic builds:

#### 1. Execution Time Limits
- **Maximum duration**: 60 minutes (default)
- **Enforced at runtime**

```yaml
steps:
  - name: build
    template: build-app
```

#### 2. Resource Limits
Build pipelines have resource limits as configured by the user.

```yaml
steps:
  - name: build
    resources:
      limits:
        memory: "4Gi"  # Within Build limits as configured by the user
        cpu: "2000m"
```

#### 3. Required Sections
Build pipelines must include:

- At least one build step. Ex: `choreo/buildpack-build@v1`

#### 4. Automatic Features
Build pipelines automatically get:
- Source code checkout
- Artifact uploading

### Build Pipeline Structure

```yaml
steps:
  # Automatic: Source checkout happens here
  - name: Install Dependencies
    inlineScript: |
      #!/bin/bash
      npm build
    resources:
      limits:
        memory: "2Gi"  # Build constraint
        cpu: "1000m"

  - name: Build
    template: choreo/buildpack-build@v1

  - name: Test
    inlineScript: |
      #!/bin/bash
      npm test -- --coverage
    outputs:
      artifacts:
        - path: coverage/
          type: test-coverage
        - path: test-results.xml
          type: test-results

  - name: Security Scan
    template: choreo/trivy-scan@v1
    
  # Automatic: Artifacts are uploaded after pipeline
```

### Build Pipeline Example

```yaml
steps:
  - name: Build and Test
    template: choreo/buildpack-build@v1
    resources:
      limits:
        memory: "4Gi"
        cpu: "2000m"

  - name: Code Analysis
    template: sonarqube-scan

  - name: Package
    template: docker-build

templates:
      
  - name: sonarqube-scan
    inlineScript: |
      #!/bin/bash
      mvn sonar:sonar
      
  - name: docker-build
    inlineScript: |
      #!/bin/bash
      docker build -t app:$Build_COMMIT_SHA .
```

## Automation Pipelines

### Purpose
Automation pipelines are general-purpose workflows for tasks like data processing, scheduled operations, infrastructure management, and complex orchestrations.

### Characteristics

#### 1. Flexible Execution Time
- No maximum duration limit
- Support for long-running operations

```yaml
steps:
  - name: data-processing
    template: process-large-dataset
```

#### 2. Flexible Resource Allocation
No hard limits on resources:

```yaml
steps:
  - name: heavy-processing
    resources:
      requests:
        memory: "16Gi"
        cpu: "8000m"
      limits:
        memory: "32Gi"  # High memory for data processing
        cpu: "16000m"
```

#### 3. Extended Capabilities
Automation pipelines can:

- Run on-demand
- Access external systems
- Modify infrastructure
- Send notifications
- Run indefinitely (with monitoring)
- Implement complex workflows

#### 4. Trigger Options
Currently only manual trigger is supported. Planned to support scheduled trigger in the future.

### Automation Pipeline Structure

```yaml

# Persistent storage for stateful operations
volumeClaimTemplates:
  - metadata:
      name: data-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "100Gi"

steps:
  - name: Extract Data
    template: data-extractor
    resources:
      requests:
        memory: "8Gi"
        cpu: "4000m"

  - name: Process Data
    template: data-processor
    resources:
      requests:
        memory: "16Gi"
        cpu: "8000m"

  - name: Generate Reports
    template: report-generator
    
  - name: Send Notifications
    template: notify-stakeholders
```

### Automation Pipeline Example

```yaml
steps:
  - name: Initialize
    inlineScript: |
      #!/bin/bash
      echo "Starting batch processing..."
      mkdir -p /data/processing

  # Parallel processing without constraints
  - - name: Worker 1
      template: process-batch
    - name: Worker 2
      template: process-batch
    - name: Worker 3
      template: process-batch

  - name: Aggregate Results
    template: aggregate
    timeout: "2h"
    resources:
      requests:
        memory: "16Gi"

  - name: Cleanup
    template: cleanup
    continueOn:
      error: true

templates:
  - name: process-batch
    inlineScript: |
      #!/bin/bash
      echo "Worker processing..."
      # Long-running process
      process-data.sh --worker 2 \
                     --batch-size 10000
    timeout: "3h"
    resources:
      requests:
        memory: "8Gi"
        cpu: "4000m"
```

## Comparison Table

| Feature | Build Pipeline | Automation Pipeline |
|---------|------------|-------------------|
| **Purpose** | Build & Test | General Automation |
| **Max Duration** | Can be configured by the user | Can be configured by the user |
| **Max Memory** | Can be configured by the user | Can be configured by the user |
| **Max CPU** | Can be configured by the user | Can be configured by the user |
| **Triggers** | Code changes | Manual |
| **Source Checkout** | Automatic | Optional |
| **Artifacts** | Automatic upload | Manual management |
| **Environment Access** | No | No |
| **Approval Gates** | Not supported | Not supported |
| **Parallel Steps** | Unlimited | Unlimited |
| **Persistent Storage** | Clean up after pipeline completion | Clean up after pipeline completion |
| **External Systems** | Full access | Full access |
| **Scheduling** | Not supported | Not supported |
| **Long-running Tasks** | Not recommended | Recommended |

## Pipeline Type Selection Guide

Choose the appropriate pipeline type based on your use case:

### Use Build Pipeline when:
- Building and testing code
- Running on code commits/PRs
- Require deterministic builds
- Publishing build artifacts

### Use Automation Pipeline when:
- Processing data or batches
- Orchestrating complex workflows
- Implementing long-running operations
- Integrating multiple systems

## Best Practices

### Build Pipelines
1. Keep builds fast (< 15 minutes ideal)
2. Parallelize tests
3. Cache dependencies
4. Fail fast on errors
5. Minimize resource usage

### Automation Pipelines
1. Implement proper error handling
2. Add monitoring and alerting
3. Use checkpointing for long operations
4. Clean up resources
5. Document complex workflows

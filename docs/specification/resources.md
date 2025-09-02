# Resources API Reference

## Overview

Resource management allows you to control CPU and memory allocation for pipeline steps and templates. Proper resource configuration ensures efficient pipeline execution and prevents resource exhaustion.

## Resource Definition

```yaml
resources:
  requests:                     # Minimum guaranteed resources
    memory: <string>            # Memory request
    cpu: <string>               # CPU request
  limits:                       # Maximum allowed resources
    memory: <string>            # Memory limit
    cpu: <string>               # CPU limit
```

## Resource Units

### Memory Units

Memory is specified in bytes using standard Kubernetes units:

| Unit | Name | Bytes | Example |
|------|------|-------|---------|
| `Ki` | Kibibyte | 1024 | `512Ki` |
| `Mi` | Mebibyte | 1024²  | `256Mi` |
| `Gi` | Gibibyte | 1024³ | `4Gi` |
| `Ti` | Tebibyte | 1024⁴ | `1Ti` |

**Note:** You can also use decimal units (K, M, G, T) where 1K = 1000 bytes.

### CPU Units

CPU resources are measured in CPU units:

| Unit | Description | Example | Meaning |
|------|-------------|---------|---------|
| `m` | Millicores | `500m` | 0.5 CPU core |
| (none) | Cores | `2` | 2 CPU cores |

**Conversion:** 1 CPU = 1000m (millicores)

## Resource Specifications

### Requests vs Limits

- **Requests**: Minimum guaranteed resources for the container
- **Limits**: Maximum resources the container can use

```yaml
resources:
  requests:
    memory: "256Mi"    # Guaranteed 256 MiB
    cpu: "250m"        # Guaranteed 0.25 CPU cores
  limits:
    memory: "512Mi"    # Can use up to 512 MiB
    cpu: "500m"        # Can use up to 0.5 CPU cores
```

### Behavior

- If only **limits** are specified, requests default to limits
- If only **requests** are specified, no upper bound is enforced
- Container is throttled if it exceeds CPU limit
- Container is terminated if it exceeds memory limit

## Resource Configuration Levels

### Step-Level Resources

```yaml
steps:
  - name: build
    inlineScript: |
      #!/bin/bash
      npm run build
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "2Gi"
        cpu: "2000m"
```

### Template-Level Resources

```yaml
templates:
  - name: test-runner
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      npm test
```

### Container-Level Resources

```yaml
containerSet:
  containers:
    - name: database
      image: postgres:13
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
```

## Common Resource Profiles

### Minimal Resources

For lightweight tasks:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "100m"
  limits:
    memory: "128Mi"
    cpu: "200m"
```

### Standard Resources

For typical build/test tasks:

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

### High Performance

For resource-intensive operations:

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: "2000m"
  limits:
    memory: "8Gi"
    cpu: "4000m"
```

## Resource Optimization

### Memory Optimization

```yaml
# Node.js application with heap size control
steps:
  - name: node-build
    resources:
      limits:
        memory: "2Gi"
    env:
      - name: NODE_OPTIONS
        value: "--max-old-space-size=1536"  # 1.5GB heap
    inlineScript: |
      #!/bin/bash
      npm run build
```

### CPU Optimization

```yaml
# Parallel compilation with CPU limits
steps:
  - name: parallel-build
    resources:
      requests:
        cpu: "2000m"  # Request 2 cores
      limits:
        cpu: "4000m"  # Allow burst to 4 cores
    inlineScript: |
      #!/bin/bash
      make -j4  # Use 4 parallel jobs
```

## Examples by Use Case

### Language-Specific Examples

#### Node.js Application

```yaml
steps:
  - name: node-build
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      npm ci
      npm run build
```

#### Java Application

```yaml
steps:
  - name: java-build
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "2Gi"
        cpu: "2000m"
    env:
      - name: JAVA_OPTS
        value: "-Xmx1536m -Xms512m"
    inlineScript: |
      #!/bin/bash
      mvn clean package
```

#### Python ML Pipeline

```yaml
steps:
  - name: ml-training
    resources:
      requests:
        memory: "8Gi"
        cpu: "4000m"
      limits:
        memory: "16Gi"
        cpu: "8000m"
    inlineScript: |
      #!/bin/bash
      python train_model.py
```

#### Go Application

```yaml
steps:
  - name: go-build
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      go build -o app ./cmd/main.go
```

### Task-Specific Examples

#### Docker Build

```yaml
steps:
  - name: docker-build
    resources:
      requests:
        memory: "2Gi"
        cpu: "1000m"
      limits:
        memory: "4Gi"
        cpu: "2000m"
    inlineScript: |
      #!/bin/bash
      docker build -t myapp:latest .
```

#### Database Migration

```yaml
steps:
  - name: db-migration
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    inlineScript: |
      #!/bin/bash
      npm run migrate
```

#### Security Scanning

```yaml
steps:
  - name: security-scan
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
    template: choreo/trivy-scan@v1
```

## Resource Monitoring

### Logging Resource Usage

```yaml
steps:
  - name: resource-check
    resources:
      limits:
        memory: "1Gi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      echo "=== Resource Usage ==="
      echo "Memory:"
      free -h
      echo ""
      echo "CPU:"
      nproc
      echo ""
      echo "Disk:"
      df -h
      echo ""
      echo "Running process..."
      npm run build
```

### Resource Profiling

```yaml
templates:
  - name: profile-resources
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
    inlineScript: |
      #!/bin/bash
      # Monitor resource usage during execution
      (
        while true; do
          ps aux | grep -v grep | grep node
          sleep 1
        done
      ) &
      MONITOR_PID=$!
      
      # Run actual task
      npm test
      
      # Stop monitoring
      kill $MONITOR_PID
```

## Best Practices

### 1. Start Conservative

Begin with lower resources and increase based on monitoring:

```yaml
# Start with
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Increase if needed based on actual usage
```

### 2. Set Appropriate Limits

Always set limits to prevent runaway processes:

```yaml
resources:
  requests:
    memory: "1Gi"
  limits:
    memory: "2Gi"  # 2x request for burst capacity
```

### 3. Consider Task Requirements

Different tasks need different resources:

```yaml
# Compilation - CPU intensive
- name: compile
  resources:
    requests:
      cpu: "2000m"      # More CPU
      memory: "512Mi"   # Less memory

# Testing - Memory intensive  
- name: test
  resources:
    requests:
      cpu: "500m"       # Less CPU
      memory: "2Gi"     # More memory
```

### 4. Use Resource Classes

Define standard resource profiles:

```yaml
# Small tasks
templates:
  - name: small-task
    resources:
      requests: { memory: "128Mi", cpu: "100m" }
      limits: { memory: "256Mi", cpu: "200m" }

# Medium tasks
  - name: medium-task
    resources:
      requests: { memory: "512Mi", cpu: "500m" }
      limits: { memory: "1Gi", cpu: "1000m" }

# Large tasks
  - name: large-task
    resources:
      requests: { memory: "2Gi", cpu: "2000m" }
      limits: { memory: "4Gi", cpu: "4000m" }
```

## Troubleshooting

### Out of Memory (OOM) Errors

If a container is killed due to OOM:

1. Increase memory limit
2. Optimize application memory usage
3. Check for memory leaks

```yaml
# Fix OOM issues
resources:
  limits:
    memory: "2Gi"  # Increased from 1Gi
env:
  - name: NODE_OPTIONS
    value: "--max-old-space-size=1536"  # Tune heap size
```

### CPU Throttling

If tasks are running slowly:

1. Increase CPU limit
2. Check CPU request vs limit ratio
3. Optimize CPU-intensive operations

```yaml
# Fix CPU throttling
resources:
  requests:
    cpu: "1000m"   # Increased request
  limits:
    cpu: "2000m"   # Allow burst
```

### Resource Quotas

If pipeline fails to start:

1. Check namespace resource quotas
2. Reduce resource requests
3. Contact administrators for quota increase

## Resource Calculation Guidelines

### Memory Estimation

- **Base OS**: 50-100 Mi
- **Runtime** (Node.js, Python): 100-200 Mi
- **Application**: Varies by size
- **Buffer**: Add 20-50% overhead

### CPU Estimation  

- **Single-threaded**: 100-500m
- **Multi-threaded**: 1000-4000m
- **Compilation**: 2000-8000m
- **I/O bound**: 100-500m

## Performance Tips

1. **Request what you need** - Don't over-provision
2. **Limit appropriately** - Prevent resource exhaustion
3. **Monitor usage** - Adjust based on metrics
4. **Optimize first** - Before increasing resources
5. **Use parallelism wisely** - Balance CPU cores with parallel tasks
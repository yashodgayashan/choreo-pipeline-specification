# Best Practices Guide

## Overview

This guide provides best practices for creating efficient, maintainable, and secure pipelines using the Choreo Pipeline Specification.

## Pipeline Type Selection


### Don't Mix Concerns

**Bad Practice:**
```yaml
# Mixing concerns in one pipeline
steps:
  - name: build
    template: build-app
  - name: heavy-data-processing  # CI shouldn't run long tasks
    template: process-large-dataset
```

**Good Practice:**
```yaml
# Focused CI pipeline
steps:
  - name: build
    template: build-app
  - name: test
    template: run-tests
```

```yaml
# Separate automation pipeline for data processing
steps:
  - name: process-data
    template: process-large-dataset
```

## Resource Management

### Right-Size Resources

Start small and increase based on monitoring:

✅ **Good Practice:**
```yaml
steps:
  - name: build
    resources:
      requests:
        memory: "512Mi"  # Start conservative
        cpu: "500m"
      limits:
        memory: "1Gi"    # 2x request for burst
        cpu: "1000m"
```

### Use Resource Classes

Define standard resource profiles:

```yaml
templates:
  # Small tasks
  - name: small-task
    resources:
      requests: { memory: "256Mi", cpu: "250m" }
      limits: { memory: "512Mi", cpu: "500m" }
  
  # Medium tasks
  - name: medium-task
    resources:
      requests: { memory: "1Gi", cpu: "1000m" }
      limits: { memory: "2Gi", cpu: "2000m" }
  
  # Large tasks
  - name: large-task
    resources:
      requests: { memory: "4Gi", cpu: "4000m" }
      limits: { memory: "8Gi", cpu: "8000m" }
```

## Security Best Practices

### Never Hardcode Secrets

**Bad Practice:**
```yaml
env:
  - name: API_KEY
    value: "sk-1234567890abcdef"  # Never do this!
```

**Good Practice:**
```yaml
env:
  - name: API_KEY
    value: "{{SECRETS.API_KEY}}"  # Use Choreo secrets
```

### Principle of Least Privilege

Grant minimal required permissions:

```yaml
steps:
  - name: read-only-task
    volumeMounts:
      - name: data
        mountPath: /data
        readOnly: true  # Read-only when write not needed
```

### Scan for Vulnerabilities

Always include security scanning:

```yaml
steps:
  - name: build
    template: build-app
  
  - name: security-scan
    template: choreo/trivy-scan@v1
```

## Performance Optimization

### Parallelize When Possible

Run independent tasks in parallel:

✅ **Good Practice:**
```yaml
steps:
  - name: setup
    template: setup
  
  # Run tests in parallel
  - - name: unit-tests
      template: unit-test
    - name: integration-tests
      template: integration-test
    - name: lint
      template: lint-check
  
  - name: report
    template: test-report
```

### Cache Dependencies

Use volumes for caching:

```yaml
volumes:
  - name: npm-cache
    emptyDir:
      sizeLimit: "2Gi"

steps:
  - name: install
    volumeMounts:
      - name: npm-cache
        mountPath: /root/.npm
    inlineScript: |
      #!/bin/bash
      npm ci --cache /root/.npm
```

### Optimize Docker Builds

Use build cache and multi-stage builds:

```yaml
steps:
  - name: docker-build
    inlineScript: |
      #!/bin/bash
      # Use cache from previous builds
      docker build \
        --cache-from myapp:latest \
        --target production \
        -t myapp:new .
```

## Error Handling

### Use Retry Strategies

Add retries for flaky operations:

```yaml
steps:
  - name: network-operation
    template: api-call
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
      backoff:
        duration: "30s"
        factor: 2
        maxDuration: "5m"
```

### Handle Non-Critical Failures

Use `continueOn` for optional steps:

```yaml
steps:
  - name: critical-build
    template: build
  
  - name: optional-notification
    template: send-slack
    continueOn:
      error: true  # Don't fail pipeline if notification fails
```

## Template Design

### Create Reusable Templates

Design templates for reusability:

**Good Practice:**
```yaml
templates:
  - name: node-test
    inputs:
      parameters:
        - name: test-command
          value: "test"  # Default value
        - name: node-version
          value: "18"
    inlineScript: |
      #!/bin/bash
      npm run {{inputs.parameters.test-command}}
    image: "node:{{inputs.parameters.node-version}}-alpine"
```

### Keep Templates Focused

Single responsibility per template:

```yaml
templates:
  # Good: Focused templates
  - name: install-deps
    inlineScript: |
      #!/bin/bash
      npm ci
  
  - name: run-tests
    inlineScript: |
      #!/bin/bash
      npm test
  
  - name: build-app
    inlineScript: |
      #!/bin/bash
      npm run build
```

## Pipeline Organization

### Use Clear Naming

Use descriptive, consistent names:

```yaml
steps:
  # Good naming
  - name: install-dependencies
  - name: run-unit-tests
  - name: run-integration-tests
  - name: build-docker-image
  - name: push-to-registry
  
  # Bad naming
  - name: step1
  - name: test
  - name: build
```

### Add Comments

Document complex logic:

```yaml
steps:
  # Extract data from multiple sources and merge
  - name: data-extraction
    template: extract
    arguments:
      parameters:
        # Use yesterday's date for processing
        - name: date
          value: "{{workflow.parameters.date}}"
```

### Group Related Steps

Organize steps logically:

```yaml
steps:
  # === Setup Phase ===
  - name: checkout-code
    template: git-clone
  - name: install-dependencies
    template: npm-install
  
  # === Quality Checks (Parallel) ===
  - - name: lint
      template: eslint
    - name: security-scan
      template: security-check
    - name: license-check
      template: license-scan
  
  # === Build Phase ===
  - name: compile
    template: build-app
  - name: package
    template: docker-build
```

## CI Pipeline Best Practices

### Keep Builds Fast

Target < 15 minutes for CI:

```yaml
pipelineType: ci
steps:
  # Parallelize to reduce time
  - - name: quick-lint
    - name: unit-tests
  
  - name: build
```

### Fail Fast

Exit early on critical failures:

```yaml
steps:
  - name: syntax-check
    template: check-syntax
    # No retry - fail immediately
  
  - name: build
    template: build-app
    # Only proceed if syntax is valid
```

## Automation Pipeline Best Practices

### Implement Checkpointing

Save progress for long operations:

```yaml
steps:
  - name: process-batch-1
    template: process
    outputs:
      parameters:
        - name: checkpoint
          valueFrom:
            path: /tmp/checkpoint.txt
  
  - name: process-batch-2
    template: process
    arguments:
      parameters:
        - name: start-from
          value: "{{steps.process-batch-1.outputs.parameters.checkpoint}}"
```

### Add Monitoring

Include monitoring and alerting:

```yaml
steps:
  - name: main-process
    template: process-data
  
  - name: check-metrics
    inlineScript: |
      #!/bin/bash
      # Check process metrics
      ERROR_COUNT=$(cat /tmp/errors.log | wc -l)
      if [ $ERROR_COUNT -gt 10 ]; then
        # Send alert
        curl -X POST $ALERT_WEBHOOK \
          -d "High error count: $ERROR_COUNT"
        exit 1
      fi
```

## Testing Best Practices

### Test at Multiple Levels

```yaml
steps:
  # Unit tests - fast, isolated
  - name: unit-tests
    template: unit-test
  
  # Integration tests - slower, dependencies
  - name: integration-tests
    template: integration-test
  
  # E2E tests - slowest, full system
  - name: e2e-tests
    template: e2e-test
```

### Generate Test Reports

Capture and store test results:

```yaml
steps:
  - name: run-tests
    inlineScript: |
      #!/bin/bash
      npm test -- --coverage --json > test-results.json
    outputs:
      artifacts:
        - path: test-results.json
          type: test-results
        - path: coverage/
          type: coverage-report
```

## Maintenance

### Regular Reviews

Schedule pipeline reviews:
- Monthly: Review resource usage
- Quarterly: Update dependencies
- Annually: Major refactoring

### Monitor Pipeline Metrics

Track key metrics:
- Execution time
- Success rate
- Resource utilization
- Cost

## Common Pitfalls to Avoid

### 1. Over-Engineering

**Avoid:**
- Excessive parallelization
- Too many small steps

**Instead:**
- Keep it simple
- Combine related tasks
- Parallelize only when beneficial

### 2. Ignoring Limits

**Avoid:**
- Exceeding CI pipeline limits
- Unlimited resource requests
- No timeouts

**Instead:**
- Respect pipeline type constraints
- Set reasonable limits
- Always set timeouts

### 3. Poor Secret Management

**Avoid:**
- Hardcoded credentials
- Secrets in logs
- Shared secrets across environments

**Instead:**
- Use Choreo secrets
- Mask sensitive output
- Environment-specific secrets

## Summary Checklist

- [ ] Choose appropriate pipeline type
- [ ] Set resource limits
- [ ] Never hardcode secrets
- [ ] Add error handling
- [ ] Include security scanning
- [ ] Implement monitoring
- [ ] Use caching where possible
- [ ] Document complex logic
- [ ] Test thoroughly
- [ ] Review regularly

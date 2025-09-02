# Migration Guide: GitHub Actions to Choreo Pipelines

## Overview

This guide helps you migrate from GitHub Actions workflows to Choreo Pipeline Specification. It covers workflow structure translation, action replacements, and migration strategies.

## Key Differences

| Feature | GitHub Actions | Choreo |
|---------|---------------|--------|
| Configuration Language | YAML | YAML |
| File Location | `.github/workflows/` | Choreo Console Configuration |
| Execution Model | Jobs and Steps | Steps and Templates |
| Marketplace | GitHub Actions Marketplace | Choreo Templates |
| Runners | GitHub-hosted/Self-hosted | Container-based |
| Secrets | GitHub Secrets | Choreo Secrets/Variables |
| Matrix Builds | Built-in matrix strategy | Parallel steps or loops |

## Workflow Structure Comparison

### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build application
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to production
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          npm run deploy:prod
```

### Equivalent Choreo Pipeline
```yaml
# Use volumeClaimTemplates for persistent storage (artifacts not supported yet)
volumeClaimTemplates:
  - metadata:
      name: build-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  # Build and test steps
  - name: Install Dependencies with Cache
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Use cached dependencies if available
      if [ -d "/cache/node_modules" ]; then
        echo "Restoring cached dependencies..."
        cp -r /cache/node_modules ./
      fi

      # Install dependencies
      npm ci

      # Cache dependencies
      mkdir -p /cache
      cp -r node_modules /cache/
    volumeMounts:
      - name: build-cache
        mountPath: /cache

  - name: Run tests
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm test

  - name: Build application
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm run build

      # Store build artifacts in persistent storage
      # Artifact management is not currently supported
      # Using persistent volume as workaround
      mkdir -p /cache/dist
      cp -r dist/ /cache/
    volumeMounts:
      - name: build-cache
        mountPath: /cache

  - name: Deploy to production
    when: "{{CI_BRANCH}} == 'main'"
    env:
      - name: API_KEY
        value: "{{SECRETS.API_KEY}}"
      - name: NODE_VERSION
        value: "{{VARIABLES.NODE_VERSION}}"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Retrieve build artifacts from persistent storage
      if [ -d "/cache/dist" ]; then
        cp -r /cache/dist ./
      fi

      npm run deploy:prod
    volumeMounts:
      - name: build-cache
        mountPath: /cache
```

## Common GitHub Actions Patterns to Choreo

### 1. Triggers and Events

**GitHub Actions:**
```yaml
on:
  push:
    branches: [main, develop]
    paths:
      - 'src/**'
      - 'package.json'
  pull_request:
    types: [opened, synchronize]
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
```

**Choreo:**
```yaml
# Cron scheduling is not currently supported
# Manual triggers are handled through Choreo Console

# Use environment variables for configuration
env:
  - name: ENVIRONMENT
    value: "{{VARIABLES.ENVIRONMENT}}"  # Set via Choreo Console
```

**Migration Notes:**

- **Scheduled workflows**: Not currently supported, consider external schedulers
- **Path filters**: Implement logic within pipeline steps if needed
- **Manual triggers**: Use Choreo Console to trigger pipelines
- **Branch-based triggers**: Choreo pipelines are triggered through the platform

### 2. Matrix Strategy

**GitHub Actions:**
```yaml
strategy:
  matrix:
    node: [14, 16, 18]
    os: [ubuntu-latest, windows-latest, macos-latest]
    
runs-on: ${{ matrix.os }}

steps:
  - uses: actions/setup-node@v3
    with:
      node-version: ${{ matrix.node }}
  - run: npm test
```

**Choreo:**
```yaml
# Parallel execution for different versions
steps:
  - - name: Test Node 14
      container:
        image: node:14-alpine
      inlineScript: |
        #!/bin/bash
        npm test
    
    - name: Test Node 16
      container:
        image: node:16-alpine
      inlineScript: |
        #!/bin/bash
        npm test
    
    - name: Test Node 18
      container:
        image: node:18-alpine
      inlineScript: |
        #!/bin/bash
        npm test
```

### 3. Caching

**GitHub Actions:**
```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

**Choreo:**
```yaml
# Use volumeClaimTemplates for persistent caching
volumeClaimTemplates:
  - metadata:
      name: node-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  - name: Install Dependencies with Cache
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Use cached node_modules if available
      if [ -d "/cache/node_modules" ]; then
        echo "Restoring cached dependencies..."
        cp -r /cache/node_modules ./
      fi

      # Install dependencies
      npm ci

      # Update cache
      echo "Caching dependencies..."
      mkdir -p /cache
      cp -r node_modules /cache/
    volumeMounts:
      - name: node-cache
        mountPath: /cache
```

### 4. Container Services

**GitHub Actions:**
```yaml
services:
  postgres:
    image: postgres:14
    env:
      POSTGRES_PASSWORD: postgres
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
    ports:
      - 5432:5432

  redis:
    image: redis:7
    options: >-
      --health-cmd "redis-cli ping"
      --health-interval 10s
    ports:
      - 6379:6379
```

**Choreo:**
```yaml
steps:
  - name: Integration Tests
    containerSet:
      containers:
        - name: tests
          image: node:18-alpine
          env:
            - name: DATABASE_URL
              value: "postgres://postgres:postgres@localhost:5432/test"
            - name: REDIS_URL
              value: "redis://localhost:6379"
        
        - name: postgres
          image: postgres:14
          env:
            - name: POSTGRES_PASSWORD
              value: "postgres"
          readinessProbe:
            exec:
              command: ["pg_isready"]
            periodSeconds: 10
            timeoutSeconds: 5
        
        - name: redis
          image: redis:7
          readinessProbe:
            exec:
              command: ["redis-cli", "ping"]
            periodSeconds: 10
```

### 5. Conditionals and Expressions

**GitHub Actions:**
```yaml
- name: Deploy to staging
  if: github.event_name == 'push' && contains(github.ref, 'staging')
  run: npm run deploy:staging

- name: Deploy to production
  if: |
    github.event_name == 'push' &&
    github.ref == 'refs/heads/main' &&
    contains(github.event.head_commit.message, '[deploy]')
  run: npm run deploy:production
```

**Choreo:**
```yaml
steps:
  - name: Deploy to staging
    when: "{{VARIABLES.ENVIRONMENT}} == 'staging'"
    inlineScript: |
      #!/bin/bash
      npm run deploy:staging

  - name: Deploy to production
    when: "{{VARIABLES.ENVIRONMENT}} == 'production'"
    inlineScript: |
      #!/bin/bash
      npm run deploy:production
```

**Migration Notes:**
- Use `VARIABLES` instead of GitHub context for conditional logic
- Configure deployment conditions through Choreo Console variables

## Action Migration Guide

### Common GitHub Actions to Choreo Templates

| GitHub Action | Choreo Solution | Usage |
|--------------|-----------------|--------|
| `actions/checkout@v3` | Automatic in CI pipelines | Source checkout |
| `actions/setup-node@v3` | Container with Node image | Node.js environment |
| `actions/cache@v3` | `volumeClaimTemplates` | Dependency caching |
| `actions/upload-artifact@v3` | Persistent volumes (artifacts not supported yet) | Artifact storage |
| `actions/download-artifact@v3` | Persistent volumes (artifacts not supported yet) | Artifact retrieval |
| `docker/build-push-action@v4` | `choreo/docker-build@v1` | Docker builds |
| `docker/login-action@v2` | Built into docker template | Registry auth |
| `actions/github-script@v6` | Inline scripts with GitHub CLI | GitHub API calls |
| `slackapi/slack-github-action@v1` | Inline scripts with curl/webhook | Slack notifications |
| `aws-actions/configure-aws-credentials@v2` | Environment variables | AWS auth |

### Custom Action Migration

**GitHub Actions Custom Action:**
```yaml
# .github/actions/custom-deploy/action.yml
name: 'Custom Deploy'
inputs:
  environment:
    required: true
  version:
    required: true
runs:
  using: 'composite'
  steps:
    - run: |
        echo "Deploying ${{ inputs.version }} to ${{ inputs.environment }}"
        ./scripts/deploy.sh ${{ inputs.environment }} ${{ inputs.version }}
      shell: bash

# Usage
- uses: ./.github/actions/custom-deploy
  with:
    environment: production
    version: v1.2.3
```

**Choreo Approach:**
```yaml
# Custom templates with arguments are not currently supported
# Use inline scripts with environment variables instead

steps:
  - name: Deploy
    env:
      - name: DEPLOY_ENVIRONMENT
        value: "{{VARIABLES.DEPLOY_ENVIRONMENT}}"
      - name: DEPLOY_VERSION
        value: "{{VARIABLES.DEPLOY_VERSION}}"
    inlineScript: |
      #!/bin/bash
      echo "Deploying $DEPLOY_VERSION to $DEPLOY_ENVIRONMENT"
      ./scripts/deploy.sh "$DEPLOY_ENVIRONMENT" "$DEPLOY_VERSION"
```

## Composite Workflows Migration

**GitHub Actions Reusable Workflow:**
```yaml
# .github/workflows/reusable-build.yml
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '18'
    secrets:
      npm-token:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.npm-token }}

# Usage in another workflow
jobs:
  call-build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '16'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

**Choreo Approach:**
```yaml
# Reusable workflows/templates with arguments are not currently supported
# Use inline scripts with environment variables for reusable logic

steps:
  - name: Reusable Build
    containerSet:
      - name: builder
        image: "node:{{VARIABLES.NODE_VERSION}}-alpine"
    env:
      - name: NPM_TOKEN
        value: "{{SECRETS.NPM_TOKEN}}"
      - name: NODE_VERSION
        value: "{{VARIABLES.NODE_VERSION}}"
    inlineScript: |
      #!/bin/bash
      echo "Building with Node.js $NODE_VERSION"
      npm ci
      npm run build
```


## Best Practices for Migration

### 1. Analyze Your Workflows
- Inventory all GitHub Actions workflows
- Identify reusable workflows and composite actions
- Document action marketplace dependencies
- Map secrets and environment variables

### 2. Choose Pipeline Types
```yaml
# CI Pipeline for builds and tests
# - 60-minute timeout limit
# - Automatic source checkout
# - Built-in caching via volumeClaimTemplates

# Automation for long-running tasks
# - No time limits
# - Persistent storage
# - Cron scheduling not currently supported
```

### 3. Migrate Incrementally
- Start with simple workflows
- Test in parallel with existing GitHub Actions
- Gradually migrate complex workflows
- Keep GitHub Actions as backup initially

### 4. Leverage Choreo Features
- Use built-in templates instead of actions
- Implement proper secret management
- Utilize pipeline-specific features
- Take advantage of container-based execution

### 5. Handle GitHub-Specific Features

**GitHub Context Migration:**
```yaml
# GitHub contexts should be replaced with Choreo variables
# Configure through Choreo Console instead of relying on Git context
env:
  - name: COMMIT_SHA
    value: "{{VARIABLES.COMMIT_SHA}}"
  - name: BUILD_NUMBER
    value: "{{VARIABLES.BUILD_NUMBER}}"
  - name: REPOSITORY_NAME
    value: "{{VARIABLES.REPOSITORY_NAME}}"
```

**Migration Notes:**
- GitHub event context is not available in Choreo
- Use Choreo Console variables for deployment conditions
- Implement repository information through environment variables

## Migration Checklist

- [ ] Export all GitHub Actions workflows
- [ ] Document action marketplace dependencies
- [ ] Map GitHub secrets to Choreo Secrets and Variables
- [ ] Convert workflow YAML to Choreo format
- [ ] Replace actions with inline scripts (templates with arguments not supported)
- [ ] Plan alternatives for artifact sharing (not supported yet)
- [ ] Implement caching using volumeClaimTemplates
- [ ] Plan alternatives for scheduled workflows (cron not supported)
- [ ] Test migrated pipelines
- [ ] Update CI/CD documentation
- [ ] Configure environment variables in Choreo Console
- [ ] Update team permissions

## Common Gotchas

1. **Automatic Checkout**: CI pipelines in Choreo automatically check out code
2. **No Cron Scheduling**: Scheduled workflows are not currently supported
3. **No Template Arguments**: Use environment variables instead of template parameters
4. **No Artifact Management**: Use volumeClaimTemplates for sharing files between steps
5. **GitHub Context**: GitHub-specific contexts are not available, use Choreo Variables
6. **Path Filters**: Implement file change detection in pipeline logic if needed
7. **Manual Triggers**: Configure through Choreo Console variables

## Advanced Migration Patterns

### GitHub API Integration
```yaml
steps:
  - name: GitHub API Operations
    env:
      - name: GITHUB_TOKEN
        value: "{{SECRETS.GITHUB_TOKEN}}"
    inlineScript: |
      #!/bin/bash
      # Create a release
      curl -X POST \
        -H "Authorization: token $GITHUB_TOKEN" \
        -H "Accept: application/vnd.github.v3+json" \
        https://api.github.com/repos/owner/repo/releases \
        -d '{"tag_name":"v1.0.0","name":"Release v1.0.0"}'
```

### Notification Integration
```yaml
steps:
  - name: Send Build Notification
    env:
      - name: SLACK_WEBHOOK_URL
        value: "{{SECRETS.SLACK_WEBHOOK_URL}}"
      - name: BUILD_STATUS
        value: "{{VARIABLES.BUILD_STATUS}}"
    inlineScript: |
      #!/bin/bash
      # Send Slack notification
      curl -X POST \
        -H "Content-Type: application/json" \
        -d "{\"text\":\"Build $BUILD_STATUS for {{VARIABLES.REPOSITORY_NAME}}\"}" \
        "$SLACK_WEBHOOK_URL"
```

## Support Resources

- [CI Pipeline Guide](../../specification/pipeline-types.md#ci-pipelines)
- [Best Practices Guide](../best-practices.md)

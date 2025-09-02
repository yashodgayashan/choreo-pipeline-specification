# Migration Guide: Bitbucket Pipelines to Choreo Pipelines

## Overview

This guide helps you migrate from Bitbucket Pipelines to Choreo Pipeline Specification. It covers `bitbucket-pipelines.yml` conversion, Pipes migration, and Bitbucket-specific features.

## Key Differences

| Feature | Bitbucket Pipelines | Choreo |
|---------|-------------------|--------|
| Configuration File | `bitbucket-pipelines.yml` | `Choreo Console Configuration` |
| Pipeline Structure | Steps in pipelines | Steps and inline scripts |
| Execution | Bitbucket runners | Container-based execution |
| Variables | Repository/Deployment variables | Choreo Variables/Secrets |
| Pipes | Bitbucket Pipes | Built-in templates (no arguments) |
| Artifacts | Bitbucket artifacts | Not supported yet (use volumeClaimTemplates) |
| Caching | Bitbucket caches | `volumeClaimTemplates` |

## Pipeline Structure Comparison

### Bitbucket Pipeline
```yaml
# bitbucket-pipelines.yml
image: node:18

definitions:
  caches:
    npm: ~/.npm
    nodemodules: node_modules
  
  services:
    postgres:
      image: postgres:14
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
  
  steps:
    - step: &build
        name: Build
        caches:
          - node
          - npm
        script:
          - npm ci
          - npm run build
        artifacts:
          - dist/**

    - step: &test
        name: Test
        caches:
          - node
        script:
          - npm test
        after-script:
          - echo "Tests completed"

pipelines:
  default:
    - step: *build
    - step: *test

  branches:
    main:
      - step: *build
      - parallel:
        - step:
            name: Unit Tests
            script:
              - npm run test:unit
        - step:
            name: Integration Tests
            services:
              - postgres
            script:
              - npm run test:integration
      - step:
            name: Deploy to Production
            deployment: production
            trigger: manual
            script:
              - pipe: atlassian/aws-s3-deploy:1.1.0
                variables:
                  AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
                  AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
                  AWS_DEFAULT_REGION: us-east-1
                  S3_BUCKET: my-bucket
                  LOCAL_PATH: dist

    develop:
      - step: *build
      - step: *test
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - npm run deploy:staging

  pull-requests:
    '**':
      - step: *build
      - step: *test
      - step:
          name: Security Scan
          script:
            - pipe: atlassian/git-secrets-scan:0.5.1

  custom:
    release:
      - step:
          name: Create Release
          script:
            - npm version patch
            - git push && git push --tags
```

### Equivalent Choreo Pipeline
```yaml
# Template arguments and artifact management are not supported
# Use volumeClaimTemplates for caching and file sharing
volumeClaimTemplates:
  - metadata:
      name: build-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  # Build with caching
  - name: Build with Cache
    container:
      image: node:18
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Restore cache if available
      if [ -d "/cache/node_modules" ]; then
        echo "Restoring cached dependencies..."
        cp -r /cache/node_modules ./
      fi

      # Install and build
      npm ci
      npm run build

      # Update cache
      mkdir -p /cache
      cp -r node_modules /cache/

      # Store build artifacts (artifact management not supported yet)
      mkdir -p /cache/dist
      cp -r dist/ /cache/
    volumeMounts:
      - name: build-cache
        mountPath: /cache

  - name: Run Tests
    container:
      image: node:18
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Restore dependencies from cache
      if [ -d "/cache/node_modules" ]; then
        cp -r /cache/node_modules ./
      fi

      npm test
      echo "Tests completed"
    volumeMounts:
      - name: build-cache
        mountPath: /cache

  # Parallel tests for main branch
  - - name: Unit Tests
      when: "{{VARIABLES.BRANCH}} == 'main'"
      container:
        image: node:18
      inlineScript: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        if [ -d "/cache/node_modules" ]; then
          cp -r /cache/node_modules ./
        fi
        npm run test:unit
      volumeMounts:
        - name: build-cache
          mountPath: /cache

    - name: Integration Tests
      when: "{{VARIABLES.BRANCH}} == 'main'"
      containerSet:
        - name: tests
          image: node:18
          env:
            - name: DATABASE_URL
              value: "postgres://test:test@localhost:5432/testdb"
        - name: postgres
          image: postgres:14
          env:
            - name: POSTGRES_DB
              value: "testdb"
            - name: POSTGRES_USER
              value: "test"
            - name: POSTGRES_PASSWORD
              value: "test"
      inlineScript: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        if [ -d "/cache/node_modules" ]; then
          cp -r /cache/node_modules ./
        fi
        npm run test:integration
      volumeMounts:
        - name: build-cache
          mountPath: /cache

  # Deployment steps (no manual approval support)
  - name: Production Deployment
    when: "{{VARIABLES.BRANCH}} == 'main' && {{VARIABLES.DEPLOY_PRODUCTION}} == 'true'"
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Restore build artifacts
      if [ -d "/cache/dist" ]; then
        cp -r /cache/dist ./
      fi

      aws s3 sync dist/ s3://my-bucket/ --region us-east-1 --delete
    volumeMounts:
      - name: build-cache
        mountPath: /cache

  - name: Deploy to Staging
    when: "{{VARIABLES.BRANCH}} == 'develop'"
    inlineScript: |
      #!/bin/bash
      npm run deploy:staging

  # Security scan (no template arguments)
  - name: Security Scan
    when: "{{VARIABLES.IS_PR}} == 'true'"
    template: choreo/trivy-scan@v1
    env:
      - name: SCAN_PATH
        value: "{{VARIABLES.SCAN_PATH}}"
```

## Common Bitbucket Patterns to Choreo

### 1. Step Anchors and References

**Bitbucket:**
```yaml
definitions:
  steps:
    - step: &shared-step
        name: Shared Step
        script:
          - echo "Shared logic"
          - npm install

pipelines:
  default:
    - step: *shared-step
    - step:
        <<: *shared-step
        name: Extended Step
        script:
          - echo "Additional logic"
```

**Choreo:**
```yaml
# Custom templates with arguments are not supported
# Use inline scripts for shared logic

steps:
  - name: Shared Step
    inlineScript: |
      #!/bin/bash
      echo "Shared logic"
      npm install

  - name: Extended Step
    inlineScript: |
      #!/bin/bash
      echo "Shared logic"
      npm install
      echo "Additional logic"
```

### 2. Parallel Steps

**Bitbucket:**
```yaml
pipelines:
  default:
    - parallel:
      - step:
          name: Lint
          script:
            - npm run lint
      - step:
          name: Unit Tests
          script:
            - npm run test:unit
      - step:
          name: Build
          script:
            - npm run build
```

**Choreo:**
```yaml
steps:
  - - name: Lint
      inlineScript: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        npm run lint

    - name: Unit Tests
      inlineScript: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        npm run test:unit

    - name: Build
      inlineScript: |
        #!/bin/bash
        cd $REPOSITORY_DIR
        npm run build
```

### 3. Deployment Environments

**Bitbucket:**
```yaml
pipelines:
  branches:
    main:
      - step:
          name: Deploy to Test
          deployment: test
          script:
            - echo "Deploying to test"
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - echo "Deploying to staging"
      - step:
          name: Deploy to Production
          deployment: production
          trigger: manual
          script:
            - echo "Deploying to production"
```

**Choreo:**
```yaml
# Manual approvals and deployment environments are not supported
# Use conditional logic with environment variables

steps:
  - name: Deploy to Test
    when: "{{VARIABLES.ENVIRONMENT}} == 'test'"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to test environment"
      # Add your deployment logic here

  - name: Deploy to Staging
    when: "{{VARIABLES.ENVIRONMENT}} == 'staging'"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to staging environment"
      # Add your deployment logic here

  - name: Deploy to Production
    when: "{{VARIABLES.ENVIRONMENT}} == 'production' && {{VARIABLES.DEPLOY_APPROVED}} == 'true'"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to production environment"
      # Manual approval must be handled externally
      # Add your deployment logic here
```

### 4. Pipes Migration

**Bitbucket Pipes to Choreo Templates:**

| Bitbucket Pipe | Choreo Solution |
|----------------|----------------|
| `atlassian/aws-s3-deploy` | AWS CLI in inline script |
| `atlassian/aws-cloudformation-deploy` | AWS CLI in inline script |
| `atlassian/slack-notify` | Webhook in inline script |
| `atlassian/git-secrets-scan` | `choreo/trivy-scan@v1` (no arguments) |
| `docker/docker-build-push` | `choreo/docker-build@v1` (no arguments) |
| `atlassian/npm-publish` | npm publish in inline script |
| `atlassian/scp-deploy` | scp in inline script |
| `atlassian/azure-storage-deploy` | Azure CLI in inline script |
| `sonarsource/sonarcloud-scan` | Inline script with SonarCloud CLI |

**Example Pipe Migration:**

**Bitbucket:**
```yaml
- pipe: atlassian/slack-notify:2.0.0
  variables:
    WEBHOOK_URL: $SLACK_WEBHOOK
    MESSAGE: "Deployment successful"
    
- pipe: docker/docker-build-push:2.0.0
  variables:
    IMAGE_NAME: my-app
    DOCKER_HUB_USERNAME: $DOCKER_HUB_USERNAME
    DOCKER_HUB_PASSWORD: $DOCKER_HUB_PASSWORD
    DOCKER_REGISTRY: docker.io
```

**Choreo:**
```yaml
# Template arguments are not supported
# Use inline scripts and environment variables

steps:
  - name: Slack Notification
    env:
      - name: SLACK_WEBHOOK
        value: "{{SECRETS.SLACK_WEBHOOK}}"
    inlineScript: |
      #!/bin/bash
      curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"Deployment successful"}' \
        "$SLACK_WEBHOOK"

  - name: Docker Build and Push
    template: choreo/docker-build@v1
    env:
      - name: IMAGE_NAME
        value: "{{VARIABLES.IMAGE_NAME}}"
      - name: DOCKER_USERNAME
        value: "{{VARIABLES.DOCKER_HUB_USERNAME}}"
      - name: DOCKER_PASSWORD
        value: "{{SECRETS.DOCKER_HUB_PASSWORD}}"
      - name: DOCKER_REGISTRY
        value: "{{VARIABLES.DOCKER_REGISTRY}}"
```

### 5. Caching

**Bitbucket:**
```yaml
definitions:
  caches:
    npm: ~/.npm
    nodemodules: node_modules
    docker: ~/.docker
    pip: ~/.cache/pip
    custom-cache: target/

pipelines:
  default:
    - step:
        caches:
          - npm
          - nodemodules
          - custom-cache
        script:
          - npm install
          - npm run build
```

**Choreo:**
```yaml
# Template arguments are not supported
# Use volumeClaimTemplates for persistent caching
volumeClaimTemplates:
  - metadata:
      name: npm-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  - name: Build with Multiple Caches
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Restore multiple caches if available
      if [ -d "/cache/npm" ]; then
        echo "Restoring npm cache..."
        cp -r /cache/npm ~/.npm
      fi

      if [ -d "/cache/node_modules" ]; then
        echo "Restoring node_modules cache..."
        cp -r /cache/node_modules ./
      fi

      if [ -d "/cache/target" ]; then
        echo "Restoring target cache..."
        cp -r /cache/target ./
      fi

      # Build
      npm install
      npm run build

      # Update caches
      mkdir -p /cache/npm /cache/node_modules /cache/target
      cp -r ~/.npm /cache/npm 2>/dev/null || true
      cp -r node_modules /cache/node_modules 2>/dev/null || true
      cp -r target /cache/target 2>/dev/null || true
    volumeMounts:
      - name: npm-cache
        mountPath: /cache
```

### 6. After-Script

**Bitbucket:**
```yaml
- step:
    name: Test
    script:
      - npm test
    after-script:
      - echo "Test exit code: $BITBUCKET_EXIT_CODE"
      - |
        if [ $BITBUCKET_EXIT_CODE -eq 0 ]; then
          echo "Tests passed"
        else
          echo "Tests failed"
        fi
```

**Choreo:**
```yaml
steps:
  - name: Test with Cleanup
    inlineScript: |
      #!/bin/bash
      set +e  # Don't exit on error
      npm test
      EXIT_CODE=$?
      
      # After-script equivalent
      echo "Test exit code: $EXIT_CODE"
      if [ $EXIT_CODE -eq 0 ]; then
        echo "Tests passed"
      else
        echo "Tests failed"
      fi
      
      # Always run cleanup
      echo "Cleaning up..."
      rm -rf temp/
      
      exit $EXIT_CODE
```

### 7. Size Limits and Timeouts

**Bitbucket:**
```yaml
- step:
    name: Large Build
    size: 2x  # Double resources
    max-time: 120  # 120 minutes
    script:
      - npm run build:large
```

**Choreo:**
```yaml
steps:
  - name: Large Build
    resources:
      requests:
        memory: "4Gi"  # Increased from default
        cpu: "2000m"    # Increased from default
    timeout: "120m"     # 120 minutes timeout
    inlineScript: |
      #!/bin/bash
      npm run build:large
```

## Repository Variables Migration

### Bitbucket Variables to Choreo

**Bitbucket Repository Variables:**

- Repository Settings > Repository variables
- Repository Settings > Deployments > Environment variables

**Choreo Variables:**
```yaml
# Variables (non-sensitive)
env:
  - name: NODE_ENV
    value: "{{VARIABLES.NODE_ENV}}"
  - name: API_URL
    value: "{{VARIABLES.API_URL}}"

# Secrets (sensitive)
env:
  - name: API_KEY
    value: "{{SECRETS.API_KEY}}"
  - name: DATABASE_PASSWORD
    value: "{{SECRETS.DATABASE_PASSWORD}}"
```

### Predefined Variables Mapping

| Bitbucket Variable | Choreo Equivalent |
|-------------------|-------------------|
| `$BITBUCKET_BRANCH` | `$CI_BRANCH` |
| `$BITBUCKET_REPO_SLUG` | `{{VARIABLES.PROJECT}}` |
| `$BITBUCKET_WORKSPACE` | `{{VARIABLES.ORG}}` |
| `$BITBUCKET_DEPLOYMENT_ENVIRONMENT` | Not supported |
| `$BITBUCKET_PR_ID` | Custom implementation |
| `$BITBUCKET_TAG` | Not supported |

## Custom Pipelines Migration

**Bitbucket Custom Pipelines:**
```yaml
pipelines:
  custom:
    deploy-to-prod:
      - variables:
          - name: ENVIRONMENT
            default: production
          - name: VERSION
      - step:
          name: Deploy $VERSION to $ENVIRONMENT
          script:
            - echo "Deploying version $VERSION to $ENVIRONMENT"

    run-migration:
      - step:
          name: Database Migration
          script:
            - npm run db:migrate
```

**Choreo:**
```yaml
# Template arguments are not supported
# Use environment variables configured through Choreo Console

steps:
  - name: Deploy Version
    when: "{{VARIABLES.DEPLOY_CUSTOM}} == 'true'"
    env:
      - name: ENVIRONMENT
        value: "{{VARIABLES.ENVIRONMENT}}"
      - name: VERSION
        value: "{{VARIABLES.VERSION}}"
    inlineScript: |
      #!/bin/bash
      echo "Deploying version $VERSION to $ENVIRONMENT"
      # Add your deployment logic here

  - name: Database Migration
    when: "{{VARIABLES.RUN_MIGRATION}} == 'true'"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm run db:migrate
```

## Best Practices for Migration

### 1. Start Simple
- Migrate default pipeline first
- Add branch-specific logic incrementally
- Test each migration phase

### 2. Script Reusability
```yaml
# Custom templates are not supported
# Use inline scripts with shared logic

steps:
  - name: Standard Build
    container:
      image: node:18
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Shared build logic
      npm ci
      npm run build
      npm test
```

### 3. Environment Management
```yaml
# ⚠️ Deployment environments and manual approvals are not supported
# Use conditional logic with environment variables

steps:
  - name: Deploy Application
    when: "{{VARIABLES.ENVIRONMENT}} == 'production'"
    env:
      - name: DEPLOY_ENV
        value: "{{VARIABLES.ENVIRONMENT}}"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to $DEPLOY_ENV environment"
      # Manual approval must be handled externally
```

### 4. Resource Optimization
```yaml
# Use appropriate container resources
steps:
  - name: Large Build
    container:
      image: node:18
      resources:
        requests:
          memory: "4Gi"
          cpu: "2000m"
    timeout: 3600  # 60-minute limit for CI pipelines
    inlineScript: |
      #!/bin/bash
      npm run build:large
```

## Migration Checklist

- [ ] Export `bitbucket-pipelines.yml`
- [ ] Document repository variables
- [ ] List all Pipes dependencies
- [ ] Convert step definitions to inline scripts
- [ ] Migrate branch pipelines
- [ ] Convert PR pipelines
- [ ] Migrate custom pipelines
- [ ] Replace Pipes with inline scripts (no template arguments)
- [ ] Set up caching with volumeClaimTemplates
- [ ] Configure Choreo variables/secrets
- [ ] Implement custom deployment logic (no environment management)
- [ ] Replace manual triggers with conditional logic
- [ ] Test migrated pipelines
- [ ] Update documentation
- [ ] Train team on Choreo

## Common Gotchas

1. **Template Arguments**: Not supported - use environment variables instead
2. **Artifact Management**: Not supported yet - use volumeClaimTemplates for file sharing
3. **Manual Triggers**: Not supported - implement conditional logic with variables
4. **Deployment Environments**: Not supported - implement custom deployment logic
5. **Max Time**: Choreo CI has 60-minute limit for all pipeline types
6. **Services**: Use containerSet for service dependencies
7. **Pipes**: Most need custom implementation with inline scripts
8. **PR Triggers**: Implement with branch naming conventions
9. **After-script**: Include in main script with error handling
10. **Custom Pipelines**: No support for dynamic parameters - use Choreo Console variables

## Advanced Patterns

### Bitbucket API Integration
```yaml
steps:
  - name: Bitbucket Operations
    env:
      - name: BB_TOKEN
        value: "{{SECRETS.BITBUCKET_TOKEN}}"
    inlineScript: |
      #!/bin/bash
      # Create deployment
      curl -X POST \
        "https://api.bitbucket.org/2.0/repositories/{{VARIABLES.ORG}}/{{VARIABLES.PROJECT}}/deployments" \
        -H "Authorization: Bearer $BB_TOKEN" \
        -d '{"environment": {"name": "production"}}'
```

### Multi-Repo Pipelines
```yaml
steps:
  - name: Trigger Other Repository
    inlineScript: |
      #!/bin/bash
      # Trigger pipeline in another repository
      curl -X POST \
        "https://api.bitbucket.org/2.0/repositories/org/other-repo/pipelines" \
        -H "Authorization: Bearer {{SECRETS.BITBUCKET_TOKEN}}" \
        -d '{
          "target": {
            "ref_type": "branch",
            "type": "pipeline_ref_target",
            "ref_name": "main"
          }
        }'
```

## Support Resources

- [Choreo Pipeline Specification](../../specification/README.md)
- [Choreo Pipeline Structure](../../specification/pipeline-structure.md)
- [Pipeline Types Guide](../../specification/pipeline-types.md)
- [Best Practices Guide](../best-practices.md)

# Migration Guide: GitLab CI to Choreo Pipelines

## Overview

This guide helps you migrate from GitLab CI/CD pipelines to Choreo Pipeline Specification. It covers `.gitlab-ci.yml` conversion, runner migration, and GitLab-specific features.

## Key Differences

| Feature | GitLab CI | Choreo |
|---------|-----------|--------|
| Configuration File | `.gitlab-ci.yml` | `Choreo Console Configuration` |
| Pipeline Structure | Stages and Jobs | Steps and inline scripts |
| Runners | GitLab Runners | Container-based execution |
| Variables | GitLab CI/CD Variables | Choreo Variables/Secrets |
| Artifacts | GitLab Artifacts | Not supported yet (use volumeClaimTemplates) |
| Caching | GitLab Cache | `volumeClaimTemplates` |
| Registry | GitLab Container Registry | Any container registry |
| Dependencies | Job dependencies/needs | Step sequencing |

## Pipeline Structure Comparison

### GitLab CI Pipeline
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "18"
  DOCKER_DRIVER: overlay2

default:
  image: node:${NODE_VERSION}-alpine
  before_script:
    - npm config set registry https://registry.npmjs.org/
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  only:
    - branches
  except:
    - tags

test:unit:
  stage: test
  needs: ["build"]
  script:
    - npm run test:unit
  coverage: '/Coverage: \d+\.\d+%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  stage: test
  needs: ["build"]
  services:
    - postgres:14
    - redis:7
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: "postgresql://test:test@postgres:5432/test"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm run test:integration

deploy:staging:
  stage: deploy
  needs: ["test:unit", "test:integration"]
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - npm run deploy:staging
  only:
    - develop

deploy:production:
  stage: deploy
  needs: ["test:unit", "test:integration"]
  environment:
    name: production
    url: https://example.com
  script:
    - npm run deploy:production
  when: manual
  only:
    - main
```

### Equivalent Choreo Pipeline
```yaml
# Use volumeClaimTemplates for caching and artifact sharing
volumeClaimTemplates:
  - metadata:
      name: build-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  # Build stage with caching
  - name: Install and Build
    containerSet:
      - name: builder
        image: "node:{{VARIABLES.NODE_VERSION}}-alpine"
    inlineScript: |
      #!/bin/sh
      cd $REPOSITORY_DIR

      # Restore cache if available
      if [ -d "/cache/node_modules" ]; then
        echo "Restoring cached dependencies..."
        cp -r /cache/node_modules ./
      fi

      # Install and build
      npm config set registry https://registry.npmjs.org/
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
    env:
      - name: NODE_VERSION
        value: "{{VARIABLES.NODE_VERSION}}"

  # Test stage - parallel execution
  - - name: Unit Tests
      inlineScript: |
        #!/bin/sh
        cd $REPOSITORY_DIR

        # Restore dependencies from cache
        if [ -d "/cache/node_modules" ]; then
          cp -r /cache/node_modules ./
        fi

        npm run test:unit
      volumeMounts:
        - name: build-cache
          mountPath: /cache

    - name: Integration Tests
      containerSet:
        - name: tests
          image: "node:{{VARIABLES.NODE_VERSION}}-alpine"
          env:
            - name: DATABASE_URL
              value: "postgresql://test:test@localhost:5432/test"
            - name: REDIS_URL
              value: "redis://localhost:6379"
        - name: postgres
          image: postgres:14
          env:
            - name: POSTGRES_DB
              value: "test"
            - name: POSTGRES_USER
              value: "test"
            - name: POSTGRES_PASSWORD
              value: "test"
        - name: redis
          image: redis:7
      inlineScript: |
        #!/bin/sh
        cd $REPOSITORY_DIR

        # Restore dependencies from cache
        if [ -d "/cache/node_modules" ]; then
          cp -r /cache/node_modules ./
        fi

        npm run test:integration
      volumeMounts:
        - name: build-cache
          mountPath: /cache

  # Deployment steps (simplified - no manual approval support)
  - name: Deploy to Staging
    when: "{{VARIABLES.ENVIRONMENT}} == 'staging'"
    env:
      - name: DEPLOY_ENVIRONMENT
        value: "{{VARIABLES.ENVIRONMENT}}"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Restore build artifacts
      if [ -d "/cache/dist" ]; then
        cp -r /cache/dist ./
      fi

      echo "Deploying to staging environment"
      # Add your deployment logic here
    volumeMounts:
      - name: build-cache
        mountPath: /cache
```

## Common GitLab CI Patterns to Choreo

### 1. Rules and Conditions

**GitLab CI:**
```yaml
deploy:
  script:
    - echo "Deploying..."
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
    - if: '$CI_COMMIT_TAG'
      when: manual
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: never
    - changes:
      - Dockerfile
      - docker-compose.yml
      when: manual
```

**Choreo:**
```yaml
steps:
  - name: Deploy
    when: "{{VARIABLES.DEPLOYMENT_ENABLED}} == 'true'"
    inlineScript: |
      #!/bin/bash
      echo "Deploying..."

  - name: Deploy Tagged Release
    when: "{{VARIABLES.IS_RELEASE}} == 'true'"
    inlineScript: |
      #!/bin/bash
      echo "Deploying release version {{VARIABLES.VERSION}}"
      # Manual approval not currently supported
      # Implement deployment logic directly

  # File change detection requires custom logic
  - name: Check Docker Changes
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      if git diff --name-only HEAD~1 | grep -E "Dockerfile|docker-compose.yml"; then
        echo "true" > /tmp/docker-changed
        echo "Docker files changed"
      else
        echo "false" > /tmp/docker-changed
      fi
    env:
      - name: DOCKER_CHANGED
        value: "{{VARIABLES.DOCKER_CHANGED}}"
```

**Migration Notes:**

- GitLab rules should be replaced with Choreo Variables
- Manual approvals are not currently supported
- File change detection must be implemented in scripts

### 2. Parallel and Matrix Jobs

**GitLab CI:**
```yaml
test:
  parallel:
    matrix:
      - NODE_VERSION: ["14", "16", "18"]
        OS: ["alpine", "bullseye"]
  image: node:${NODE_VERSION}-${OS}
  script:
    - npm test
```

**Choreo:**
```yaml
steps:
  # Parallel execution of matrix combinations
  - - name: Test Node 14 Alpine
      container:
        image: node:14-alpine
      inlineScript: |
        #!/bin/sh
        npm test

    - name: Test Node 14 Bullseye
      container:
        image: node:14-bullseye
      inlineScript: |
        #!/bin/sh
        npm test

    - name: Test Node 16 Alpine
      container:
        image: node:16-alpine
      inlineScript: |
        #!/bin/sh
        npm test

    - name: Test Node 16 Bullseye
      container:
        image: node:16-bullseye
      inlineScript: |
        #!/bin/sh
        npm test

    - name: Test Node 18 Alpine
      container:
        image: node:18-alpine
      inlineScript: |
        #!/bin/sh
        npm test

    - name: Test Node 18 Bullseye
      container:
        image: node:18-bullseye
      inlineScript: |
        #!/bin/sh
        npm test
```

### 3. Dynamic Behavior

**GitLab CI:**
```yaml
generate-config:
  stage: build
  script:
    - |
      cat > child-pipeline.yml << EOF
      test-dynamic:
        script:
          - echo "Dynamic test"
      EOF
  artifacts:
    paths:
      - child-pipeline.yml

trigger-child:
  stage: test
  trigger:
    include:
      - artifact: child-pipeline.yml
        job: generate-config
    strategy: depend
```

**Choreo:**
```yaml
# Dynamic child pipelines and templates with arguments are not supported
# Use environment variables for dynamic behavior

steps:
  - name: Generate Test Configuration
    inlineScript: |
      #!/bin/bash
      # Generate list of tests to run
      echo "test1,test2,test3" > /tmp/tests.txt

  - name: Run Dynamic Tests
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Read test configuration
      TESTS=$(cat /tmp/tests.txt)

      # Run each test
      IFS=',' read -ra TEST_ARRAY <<< "$TESTS"
      for test in "${TEST_ARRAY[@]}"; do
        echo "Running dynamic test: $test"
        # Add your test logic here
      done
    env:
      - name: TEST_SUITE
        value: "{{VARIABLES.TEST_SUITE}}"
```

**Migration Notes:**

- Dynamic child pipelines are not supported
- Use inline scripts with environment variables for dynamic behavior
- Template arguments are not currently supported

### 4. Deployment Patterns

**GitLab CI:**
```yaml
deploy:staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop:staging
    auto_stop_in: 1 week
  script:
    - echo "Deploying to staging"
  only:
    - develop

stop:staging:
  stage: deploy
  environment:
    name: staging
    action: stop
  script:
    - echo "Stopping staging environment"
  when: manual
  only:
    - develop
```

**Choreo:**
```yaml
# Manual approvals and environment management are not currently supported
# Implement deployment logic directly in steps

steps:
  - name: Deploy to Staging
    when: "{{VARIABLES.ENVIRONMENT}} == 'staging'"
    env:
      - name: DEPLOY_URL
        value: "{{VARIABLES.STAGING_URL}}"
    inlineScript: |
      #!/bin/bash
      echo "Deploying to staging environment: $DEPLOY_URL"
      # Add your deployment logic here
      kubectl apply -f k8s/staging/

  - name: Cleanup Staging Environment
    when: "{{VARIABLES.CLEANUP_ENABLED}} == 'true'"
    inlineScript: |
      #!/bin/bash
      echo "Cleaning up staging environment"
      kubectl delete namespace staging-{{VARIABLES.BRANCH_NAME}}
```

**Migration Notes:**

- Environment management features are not supported
- Manual approvals are not currently supported
- Implement deployment logic directly in pipeline steps

### 5. GitLab Services

**GitLab CI:**
```yaml
test:
  services:
    - name: postgres:14
      alias: db
      variables:
        POSTGRES_DB: test_db
        POSTGRES_USER: test_user
        POSTGRES_PASSWORD: test_pass
    - docker:dind
  variables:
    DATABASE_URL: "postgres://test_user:test_pass@db:5432/test_db"
    DOCKER_HOST: tcp://docker:2375
  script:
    - docker info
    - psql $DATABASE_URL -c "SELECT 1"
```

**Choreo:**
```yaml
steps:
  - name: Integration Test
    containerSet:
      containers:
        - name: test
          image: node:18-alpine
          env:
            - name: DATABASE_URL
              value: "postgres://test_user:test_pass@localhost:5432/test_db"
            - name: DOCKER_HOST
              value: "tcp://localhost:2375"
        
        - name: postgres
          image: postgres:14
          env:
            - name: POSTGRES_DB
              value: "test_db"
            - name: POSTGRES_USER
              value: "test_user"
            - name: POSTGRES_PASSWORD
              value: "test_pass"
        
        - name: docker
          image: docker:dind
          securityContext:
            privileged: true
```

### 6. Include and Extends

**GitLab CI:**
```yaml
include:
  - local: .gitlab/ci/build.yml
  - template: Security/SAST.gitlab-ci.yml
  - project: my-group/my-project
    ref: main
    file: .gitlab-ci-template.yml

.base-job:
  before_script:
    - echo "Setup"
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure

test:
  extends: .base-job
  script:
    - npm test
```

**Choreo:**
```yaml
# Template includes and custom templates with arguments are not supported
# Use inline scripts with shared environment variables

steps:
  - name: Test with Retry
    retries: 2
    timeout: 300
    inlineScript: |
      #!/bin/bash
      # Shared setup logic
      echo "Setup"

      # Test execution
      cd $REPOSITORY_DIR
      npm test
    env:
      - name: TEST_ENVIRONMENT
        value: "{{VARIABLES.TEST_ENVIRONMENT}}"

  # For security scanning - use built-in templates without arguments
  - name: Security Scan
    template: choreo/trivy-scan@v1
    retries: 1
    timeout: 600
```

**Migration Notes:**

- GitLab includes are not supported in Choreo
- Use inline scripts for shared logic
- Built-in templates don't support custom arguments
- Implement retry logic using retries and timeout properties

## GitLab Features Migration

### Container Registry

**GitLab CI:**
```yaml
docker:build:
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

**Choreo:**
```yaml
steps:
  - name: Build and Push Docker Image
    template: choreo/docker-build@v1
    env:
      - name: DOCKER_REGISTRY
        value: "{{VARIABLES.DOCKER_REGISTRY}}"
      - name: IMAGE_NAME
        value: "{{VARIABLES.IMAGE_NAME}}"
      - name: IMAGE_TAG
        value: "{{VARIABLES.COMMIT_SHA}}"
      - name: DOCKER_USERNAME
        value: "{{VARIABLES.REGISTRY_USER}}"
      - name: DOCKER_PASSWORD
        value: "{{SECRETS.REGISTRY_PASSWORD}}"
```

### GitLab Pages

**GitLab CI:**
```yaml
pages:
  stage: deploy
  script:
    - mkdir .public
    - cp -r dist/* .public
    - mv .public public
  artifacts:
    paths:
      - public
  only:
    - main
```

**Choreo:**
```yaml
# Artifact management is not currently supported
# Deploy directly to external hosting services

steps:
  - name: Deploy Documentation
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"
      - name: S3_BUCKET
        value: "{{VARIABLES.DOCS_BUCKET}}"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Build documentation
      mkdir -p public
      cp -r dist/* public/

      # Deploy to S3 (or your preferred static hosting)
      aws s3 sync public/ s3://$S3_BUCKET/ --delete

      echo "Documentation deployed to https://$S3_BUCKET.s3.amazonaws.com"
```

### Review Apps

**GitLab CI:**
```yaml
review:
  stage: deploy
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.example.com
    on_stop: stop_review
    auto_stop_in: 2 days
  script:
    - deploy_review_app.sh
  only:
    - merge_requests

stop_review:
  stage: deploy
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  script:
    - stop_review_app.sh
  when: manual
  only:
    - merge_requests
```

**Choreo:**
```yaml
# Environment management and auto-stop features are not supported
# Implement review app logic with environment variables

steps:
  - name: Deploy Review App
    when: "{{VARIABLES.IS_REVIEW_APP}} == 'true'"
    env:
      - name: REVIEW_URL
        value: "{{VARIABLES.REVIEW_URL}}"
      - name: REVIEW_NAME
        value: "{{VARIABLES.REVIEW_NAME}}"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      echo "Deploying review app: $REVIEW_NAME"
      echo "URL will be: $REVIEW_URL"

      # Deploy review environment
      ./scripts/deploy_review_app.sh "$REVIEW_NAME"

  - name: Cleanup Review App
    when: "{{VARIABLES.CLEANUP_REVIEW}} == 'true'"
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      echo "Cleaning up review app: {{VARIABLES.REVIEW_NAME}}"
      ./scripts/stop_review_app.sh "{{VARIABLES.REVIEW_NAME}}"
```

**Migration Notes:**

- Environment management is not supported
- Manual triggers for cleanup are not supported
- Auto-stop functionality must be implemented externally

## Variable Migration

### GitLab CI Variables to Choreo

**GitLab CI Predefined Variables:**
```yaml
script:
  - echo $CI_PROJECT_NAME
  - echo $CI_COMMIT_REF_NAME
  - echo $CI_COMMIT_SHA
  - echo $CI_PIPELINE_ID
  - echo $CI_JOB_NAME
  - echo $CI_REGISTRY
  - echo $CI_MERGE_REQUEST_ID
```

**Choreo Approach:**
```yaml
# GitLab CI predefined variables are not available
# Configure necessary values through Choreo Console Variables

env:
  - name: PROJECT_NAME
    value: "{{VARIABLES.PROJECT_NAME}}"
  - name: COMMIT_SHA
    value: "{{VARIABLES.COMMIT_SHA}}"
  - name: BUILD_NUMBER
    value: "{{VARIABLES.BUILD_NUMBER}}"
  - name: REGISTRY_URL
    value: "{{VARIABLES.DOCKER_REGISTRY}}"
  - name: ENVIRONMENT
    value: "{{VARIABLES.ENVIRONMENT}}"
```

**Migration Notes:**

- GitLab CI predefined variables are not automatically available
- Configure required values through Choreo Console Variables
- Repository and commit information must be passed as variables
- Pipeline context is not automatically injected

## Best Practices for Migration

### 1. Pipeline Organization

- Convert stages to sequential steps
- Use parallel steps for concurrent jobs
- Implement templates for reusable components

### 2. Secret Management
```yaml
# Migrate GitLab CI variables
# From: GitLab CI/CD Settings > Variables
# To: Choreo Console > Secrets

env:
  - name: API_KEY
    value: "{{SECRETS.API_KEY}}"
  - name: DATABASE_URL
    value: "{{SECRETS.DATABASE_URL}}"
```

### 3. Artifact Migration
```yaml
# Artifact management is not currently supported
# Use volumeClaimTemplates for storing files between steps
volumeClaimTemplates:
  - metadata:
      name: build-artifacts
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "10Gi"

steps:
  - name: Store Build Artifacts
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Build your application
      npm run build

      # Store artifacts in persistent volume
      mkdir -p /artifacts/dist
      cp -r dist/* /artifacts/dist/
    volumeMounts:
      - name: build-artifacts
        mountPath: /artifacts

  - name: Use Build Artifacts
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Retrieve artifacts from persistent volume
      if [ -d "/artifacts/dist" ]; then
        cp -r /artifacts/dist ./
      fi

      # Use the artifacts
      ls -la dist/
    volumeMounts:
      - name: build-artifacts
        mountPath: /artifacts
```

### 4. Runner Tags to Container Selection
```yaml
# GitLab runner tags
# tags:
#   - docker
#   - linux

# Choreo container specification
container:
  image: ubuntu:22.04
  resources:
    requests:
      memory: "2Gi"
      cpu: "1000m"
```

## Migration Checklist

- [ ] Export `.gitlab-ci.yml` files
- [ ] Document GitLab CI/CD variables
- [ ] Map GitLab runners to containers
- [ ] Convert stages to Choreo steps
- [ ] Migrate job dependencies
- [ ] Replace GitLab templates
- [ ] Convert include/extends to templates
- [ ] Migrate environments configuration
- [ ] Update artifact handling
- [ ] Convert services to containerSets
- [ ] Test migrated pipelines
- [ ] Update documentation
- [ ] Configure Choreo variables/secrets
- [ ] Set up deployment environments

## Common Gotchas

1. **Auto DevOps**: No direct equivalent, build custom pipelines
2. **GitLab Pages**: Deploy to external static hosting
3. **Review Apps**: Implement with custom logic
4. **DAG Pipelines**: Use step dependencies
5. **Pipeline Schedules**: Use automation pipelines with cron

## Advanced Patterns

### Multi-Project Pipelines
```yaml
# Trigger downstream project
steps:
  - name: Trigger Downstream
    inlineScript: |
      #!/bin/bash
      # Use API to trigger another Choreo pipeline
      curl -X POST \
        -H "Authorization: Bearer {{SECRETS.CHOREO_TOKEN}}" \
        https://api.choreo.dev/pipelines/trigger \
        -d '{"project": "downstream-project", "branch": "main"}'
```

### Security Scanning
```yaml
# Template arguments are not currently supported
# Use built-in templates without arguments
steps:
  - name: Security Scan
    template: choreo/trivy-scan@v1
    env:
      - name: SCAN_TYPE
        value: "{{VARIABLES.SCAN_TYPE}}"
      - name: LANGUAGE
        value: "{{VARIABLES.LANGUAGE}}"

  - name: Dependency Check
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Install security scanning tools
      npm install -g audit-ci

      # Run security checks
      npm audit --audit-level moderate
      audit-ci --moderate
    env:
      - name: PACKAGE_MANAGER
        value: "{{VARIABLES.PACKAGE_MANAGER}}"
```

## Support Resources

- [Choreo Pipeline Specification](../../specification/README.md)
- [Choreo Pipeline Structure](../../specification/pipeline-structure.md)
- [Pipeline Types Guide](../../specification/pipeline-types.md)
- [Best Practices Guide](../best-practices.md)

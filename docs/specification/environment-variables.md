# Environment Variables API Reference

## Overview

Environment variables provide a way to configure pipeline behavior without modifying the pipeline code. Choreo supports three types of environment variables: Variables (configuration), Secrets (sensitive data), and Inline values (static).

## Variable Types

### 1. Variables (Configuration)
Non-sensitive configuration values managed through the Choreo console

### 2. Secrets (Sensitive Data)
Sensitive values securely managed through the Choreo console

### 3. Inline Values
Static values defined directly in the pipeline configuration

## Environment Variable Definition

```yaml
env:
  - name: <string>              # Required: Variable name
    value: <string>             # Required: Variable value or reference
    valueFrom: <string>         # Optional: Alternative value source
```

## Variable Reference Syntax

### Variables Reference

Variables are referenced using the `{{VARIABLES.NAME}}` syntax:

```yaml
env:
  - name: API_ENDPOINT
    value: "{{VARIABLES.API_ENDPOINT}}"
```

#### Hierarchical Variables (CI Pipelines Only)

For CI pipelines, variables can be accessed at different organizational levels:

```yaml
env:
  # Organization level variables
  - name: ORG_REGISTRY_URL
    value: "{{VARIABLES.ORG.REGISTRY_URL}}"

  # Project level variables
  - name: PROJECT_DATABASE_URL
    value: "{{VARIABLES.PROJECT.DATABASE_URL}}"

  # Component level variables (explicit)
  - name: COMPONENT_PORT
    value: "{{VARIABLES.COMPONENT.PORT}}"

  # Data Plane level variables
  - name: DT_ENDPOINT
    value: "{{VARIABLES.DT.ENDPOINT}}"
```

### Secrets Reference

Secrets are referenced using the `{{SECRETS.NAME}}` syntax:

```yaml
env:
  - name: API_TOKEN
    value: "{{SECRETS.API_TOKEN}}"
```

#### Hierarchical Secrets (CI Pipelines Only)

For CI pipelines, secrets can be accessed at different organizational levels:

```yaml
env:
  # Organization level secrets
  - name: ORG_API_KEY
    value: "{{SECRETS.ORG.API_KEY}}"

  # Project level secrets
  - name: PROJECT_DB_PASSWORD
    value: "{{SECRETS.PROJECT.DB_PASSWORD}}"

  # Component level secrets (explicit)
  - name: COMPONENT_SECRET_KEY
    value: "{{SECRETS.COMPONENT.SECRET_KEY}}"

  # Data Plane level secrets
  - name: DT_ACCESS_TOKEN
    value: "{{SECRETS.DT.ACCESS_TOKEN}}"
```


## Scope and Availability

### Step-Level Environment Variables

Environment variables defined at the step level:

```yaml
steps:
  - name: build
    env:
      - name: BUILD_ENV
        value: "production"
    inlineScript: |
      #!/bin/bash
      echo "Building in $BUILD_ENV environment"
```

### Template-Level Environment Variables

Environment variables defined in templates:

```yaml
templates:
  - name: test-template
    env:
      - name: TEST_ENV
        value: "integration"
      - name: TEST_TIMEOUT
        value: "300"
    inlineScript: |
      #!/bin/bash
      echo "Running tests in $TEST_ENV with timeout $TEST_TIMEOUT"
```

### Container-Level Environment Variables

Environment variables in container definitions:

```yaml
containerSet:
  containers:
    - name: app
      image: myapp:latest
      env:
        - name: APP_PORT
          value: "8080"
        - name: LOG_LEVEL
          value: "{{VARIABLES.LOG_LEVEL}}"
```

## Complex Variable Types

### Multi-line Values

```yaml
env:
  - name: CERTIFICATE
    value: "{{SECRETS.TLS_CERTIFICATE}}"  # Multi-line certificate
```

### Base64 Encoded Values

```yaml
env:
  - name: ENCODED_SECRET
    value: "{{SECRETS.BASE64_ENCODED_VALUE}}"
```

## Variable Precedence

When the same variable is defined at multiple levels, the following precedence applies (highest to lowest):

1. Step-level environment variables
2. Template-level environment variables
3. Global argument parameters
4. System variables

Example:

```yaml
arguments:
  parameters:
    - name: LOG_LEVEL
      value: "info"  # Lowest precedence

templates:
  - name: test
    env:
      - name: LOG_LEVEL
        value: "debug"  # Medium precedence
    inlineScript: |
      #!/bin/bash
      echo "Log level: $LOG_LEVEL"  # Will be "debug"

steps:
  - name: test-step
    template: test
    env:
      - name: LOG_LEVEL
        value: "error"  # Highest precedence - overrides template
```

## Best Practices

### 1. Naming Conventions

Use consistent naming:
- Uppercase with underscores: `API_KEY`, `DATABASE_URL`
- Prefix by type: `DB_HOST`, `API_TOKEN`, `CACHE_SIZE`
- Avoid reserved names

### 2. Security

```yaml
# Good - using secrets
env:
  - name: API_TOKEN
    value: "{{SECRETS.API_TOKEN}}"

# Bad - hardcoded secret
env:
  - name: API_TOKEN
    value: "sk-1234567890abcdef"  # Never do this!
```

### 3. Organization

Group related variables:

```yaml
env:
  # Database configuration
  - name: DB_HOST
    value: "{{VARIABLES.DB_HOST}}"
  - name: DB_PORT
    value: "{{VARIABLES.DB_PORT}}"
  - name: DB_NAME
    value: "{{VARIABLES.DB_NAME}}"
  
  # API configuration
  - name: API_URL
    value: "{{VARIABLES.API_URL}}"
  - name: API_VERSION
    value: "{{VARIABLES.API_VERSION}}"
```

### 4. Default Values

Provide sensible defaults where appropriate:

```yaml
env:
  - name: TIMEOUT
    value: "{{VARIABLES.TIMEOUT}}"
  - name: RETRY_COUNT
    value: "3"  # Default if not specified
```

## Examples

### Basic Configuration

```yaml
steps:
  - name: build
    env:
      - name: NODE_ENV
        value: "production"
      - name: BUILD_NUMBER
        value: "$BUILD_NUMBER"
    inlineScript: |
      #!/bin/bash
      echo "Building in $NODE_ENV"
      echo "Build number: $BUILD_NUMBER"
```

### Database Connection

```yaml
steps:
  - name: database-migration
    env:
      - name: DB_HOST
        value: "{{VARIABLES.DB_HOST}}"
      - name: DB_PORT
        value: "{{VARIABLES.DB_PORT}}"
      - name: DB_NAME
        value: "{{VARIABLES.DB_NAME}}"
      - name: DB_USER
        value: "{{VARIABLES.DB_USER}}"
      - name: DB_PASSWORD
        value: "{{SECRETS.DB_PASSWORD}}"
    inlineScript: |
      #!/bin/bash
      export DATABASE_URL="postgres://$DB_USER:$DB_PASSWORD@$DB_HOST:$DB_PORT/$DB_NAME"
      npm run migrate
```

### Docker Registry Authentication

```yaml
steps:
  - name: docker-push
    env:
      - name: REGISTRY_URL
        value: "{{VARIABLES.DOCKER_REGISTRY}}"
      - name: REGISTRY_USER
        value: "{{VARIABLES.DOCKER_USERNAME}}"
      - name: REGISTRY_PASS
        value: "{{SECRETS.DOCKER_PASSWORD}}"
      - name: IMAGE_NAME
        value: "{{VARIABLES.IMAGE_NAME}}"
      - name: IMAGE_TAG
        value: "{{VARIABLES.IMAGE_TAG}}"
    inlineScript: |
      #!/bin/bash
      echo "$REGISTRY_PASS" | docker login "$REGISTRY_URL" \
        --username "$REGISTRY_USER" \
        --password-stdin
      
      docker build -t "$REGISTRY_URL/$IMAGE_NAME:$IMAGE_TAG" .
      docker push "$REGISTRY_URL/$IMAGE_NAME:$IMAGE_TAG"
```

## Troubleshooting

### Common Issues

1. **Variable not found**: Ensure the variable is defined in Choreo console
2. **Secret not accessible**: Check secret permissions and scope
3. **System variable empty**: Ensure the variable is available in the current context

### Debugging Variables

```yaml
steps:
  - name: debug-env
    inlineScript: |
      #!/bin/bash
      echo "=== Environment Variables ==="
      env | sort
      echo "=== System Variables ==="
      echo "Repository: $REPOSITORY_DIR"
      echo "Branch: $BUILD_BRANCH"
      echo "Pipeline ID: $BUILD_PIPELINE_ID"
```

## Security Considerations

1. **Never hardcode secrets** in pipeline files
2. **Use secrets for sensitive data** like passwords, tokens, keys
3. **Rotate secrets regularly** through Choreo console
4. **Limit secret scope** to specific components when possible
5. **Audit variable usage** in pipeline logs
6. **Mask sensitive output** in scripts:

```yaml
inlineScript: |
  #!/bin/bash
  # Mask password in output
  echo "Connecting to database at $DB_HOST:$DB_PORT"
  # Don't echo $DB_PASSWORD
```

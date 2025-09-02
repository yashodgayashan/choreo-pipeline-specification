# Quick Start Guide

## Overview

This guide will help you create your first pipeline using the Choreo Pipeline Specification. We'll cover the basics and provide examples for different pipeline types.

## Prerequisites

Before you begin, ensure you have:

- A Choreo account and component if you need to create a build pipeline.
- Basic understanding of YAML syntax

## Step 1: Choose Your Pipeline Type

Determine which pipeline type fits your needs:

| Pipeline Type | Use Case | Constraints |
|--------------|----------|-------------|
| **CI** | Build and test code | 60 min default limit, fully flexible, 10GB default storage |
| **Automation** | General tasks, data processing | 60 min default limit, fully flexible, 10GB default storage |

## Step 2: Create Pipeline File

Configure your pipeline in the Choreo Console:

### CI Pipeline

1. **Open Choreo Console** - Navigate to your component
2. **Access Pipeline Settings** - Go to organization DevOps CI Pipelines
3. **Create Pipeline** - Click on "Create Pipeline"
    - **Plain Text**: Paste YAML directly in the console
    - **Repository**: Point to a YAML file in your repository
4. **Configure Pipeline** - Configure the pipeline as per your needs
5. **Save Pipeline** - Save the pipeline

### Automation Pipeline

1. **Open Choreo Console** - Navigate to your component
2. **Access Pipeline Settings** - Go to organization DevOps Automation Pipelines
3. **Choose Configuration Method**:
    - **Plain Text**: Paste YAML directly in the console
    - **Repository**: Point to a YAML file in your repository
4. **Save Pipeline** - Save the pipeline

## Step 3: Basic Pipeline Structure

### CI Pipeline Quick Start

In CI pipelines, Choreo will automatically checkout the repository and set the following environment variables:
- `$REPOSITORY_DIR` - Where your repository is checked out
- `$WORKSPACE` - Working directory for pipeline operations
- `$IMAGE_NAME` - The name of the built container image

Choreo will push the built image to the container registry of the organization.

Here is a basic pipeline configuration:

```yaml

steps:
  - name: Build Application
    template: choreo/buildpack-build@v1

  - name: Run Tests
    inlineScript: |
      #!/bin/bash
      echo "Running tests..."
      cd $REPOSITORY_DIR
      npm test
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "4Gi"
        cpu: "2000m"
```

### Automation Pipeline Quick Start

In Automation pipelines, Choreo will automatically set the following environment variables:
- `$WORKSPACE` - Working directory for pipeline operations

Here is a basic pipeline configuration:

```yaml
steps:
  - name: Process Data
    inlineScript: |
      #!/bin/bash
      echo "Processing daily data..."
      python process_data.py

  - name: Generate Report
    inlineScript: |
      #!/bin/bash
      echo "Generating report..."
      python generate_report.py

  - name: Send Notifications
    template: email-report

templates:
  - name: email-report
    inlineScript: |
      #!/bin/bash
      echo "Sending report to $EMAIL"
      echo "Report: $REPORT"
      email -s "Report" $EMAIL < $REPORT
    env:
      - name: EMAIL
        value: "{{VARIABLES.EMAIL}}"
      - name: REPORT
        value: "{{steps.generate-report.outputs.parameters.report}}"
```

## Step 4: Add Environment Variables

Configure variables and secrets through Choreo console, then reference them:

```yaml
steps:
  - name: build
    env:
      - name: API_URL
        value: "{{VARIABLES.API_URL}}"
      - name: API_KEY
        value: "{{SECRETS.API_KEY}}"
    inlineScript: |
      #!/bin/bash
      echo "Using API at $API_URL"
```

## Step 5: Configure Resources

```yaml
steps:
  - name: build
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "4Gi"
        cpu: "2000m"
```

## Step 6: Add Parallel Execution

Run steps in parallel using double-dash syntax:

```yaml
steps:
  - name: setup
    template: setup-environment

  # These run in parallel
  - - name: test-unit
      template: unit-tests
    - name: test-integration
      template: integration-tests
    - name: lint
      template: code-lint

  - name: report
    template: generate-report
```

## Step 7: Use Templates

Create reusable templates:

```yaml
steps:
  - name: build
    template: node-build

templates:
  - name: node-build
    inlineScript: |
      #!/bin/bash
      npm ci
      npm run build
      npm test
    image: "node:18-alpine"
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
```

## Step 8: Handle Errors

Add retry strategies and error handling:

```yaml
steps:
  - name: flaky-test
    template: integration-test
    retryStrategy:
      limit: 3
      retryPolicy: OnFailure
      backoff:
        duration: "30s"
        factor: 2
        maxDuration: "5m"

  - name: cleanup
    template: cleanup-resources
    continueOn:
      error: true  # Continue even if this fails
```

## Step 9: Validate Your Pipeline

Before deploying, validate your pipeline configuration using our JSON schema:

### Option 1: NPM Validation (Recommended)

```bash
# Install the validation tool
npm install -g ajv-cli ajv-formats

# Validate your pipeline.yaml file
ajv validate -s docs/specification/pipeline-schema.json -d pipeline.yaml --strict=false
```

### Option 2: Python Validation

```bash
# Install required packages
pip install jsonschema pyyaml

# Create validation script (validate.py)
cat > validate.py << 'EOF'
#!/usr/bin/env python3
import json
import yaml
import sys
from jsonschema import validate, ValidationError

def validate_pipeline(pipeline_file, schema_file):
    # Load schema
    with open(schema_file, 'r') as f:
        schema = json.load(f)
    
    # Load pipeline YAML
    with open(pipeline_file, 'r') as f:
        pipeline = yaml.safe_load(f)
    
    try:
        validate(instance=pipeline, schema=schema)
        print(f" {pipeline_file} is valid")
        return True
    except ValidationError as e:
        print(f" {pipeline_file} is invalid:")
        print(f" Error: {e.message}")
        print(f" Path: {' -> '.join(str(p) for p in e.path)}")
        return False

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python validate.py <pipeline.yaml> <schema.json>")
        sys.exit(1)
    
    pipeline_file = sys.argv[1]
    schema_file = sys.argv[2]
    
    if validate_pipeline(pipeline_file, schema_file):
        sys.exit(0)
    else:
        sys.exit(1)
EOF

# Run validation
python validate.py pipeline.yaml docs/specification/pipeline-schema.json
```

### Option 3: VS Code Integration

Add to your `.vscode/settings.json` for real-time validation:

```json
{
  "yaml.schemas": {
    "./docs/specification/pipeline-schema.json": ["pipeline.yaml", "*.pipeline.yaml"]
  }
}
```

### What Gets Validated:

1. **Required fields**: `steps` array must be present
2. **Step structure**: Each step needs `name` and one of `template`, `inlineScript`, or `containerSet`
3. **Resource formats**: Memory ("256Mi", "2Gi") and CPU ("500m", "2000m") patterns
4. **Environment variables**: Proper naming convention (`^[A-Z_][A-Z0-9_]*$`)
5. **Timeout formats**: Duration strings ("10s", "5m", "2h")
6. **Parallel steps**: Proper double-dash array structure

### Common Validation Errors:

**Missing required field:**
```
Error: 'steps' is a required property
Fix: Add a steps array with at least one step
```

**Invalid resource format:**
```
Error: /resources/requests/memory must match pattern "^[0-9]+[KMG]i?$"
Fix: Use "256Mi" instead of "256MB"
```

**Invalid environment variable name:**
```
Error: /env/0/name must match pattern "^[A-Z_][A-Z0-9_]*$"
Fix: Use "MY_VAR" instead of "my-var"
```

**Step missing execution method:**
```
Error: Step must have one of: template, inlineScript, containerSet
Fix: Add template: "my-template" or inlineScript: "#!/bin/bash\necho 'hello'"
```

### Pre-commit Hook (Optional)

Add validation to your git workflow:

```bash
# Create .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: validate-pipeline
        name: Validate Pipeline YAML
        entry: ajv validate -s docs/specification/pipeline-schema.json -d
        language: system
        files: pipeline\.yaml$
        pass_filenames: true
```

### Schema Details

The schema validates:

- **Choreo-specific syntax**: Templates like `choreo/buildpack-build@v1`
- **Environment variables**: `{{VARIABLES.NAME}}` and `{{SECRETS.NAME}}` patterns
- **Parallel execution**: Double-dash array syntax for parallel steps
- **Input/output parameters**: Step parameter passing
- **Volume configurations**: Persistent volumes and mounts
- **Container sets**: Multi-container step definitions

For complete schema documentation, see: [Schema Validation Guide](../specification/schema-validation.md)

## Troubleshooting

### Pipeline Not Running
- Check configuration in Choreo Console pipeline settings
- Validate YAML syntax
- Ensure pipeline type is specified

### Resource Errors
- Check resource requests vs limits
- Monitor actual usage

### Step Failures
- Check logs in Choreo console
- Add retry strategies for flaky steps
- Use `continueOn` for non-critical steps

### Variable Not Found
- Define in Choreo console first
- Check variable name and syntax
- Use correct prefix: `{{VARIABLES.NAME}}` or `{{SECRETS.NAME}}`

## Next Steps

- Explore [advanced examples](../examples/README.md)
- Read [best practices guide](./best-practices.md)
- Review [API reference](../specification/README.md)

## Getting Help

- Check the [troubleshooting guide](./troubleshooting.md)
- Review [examples](../examples/README.md)
- Contact Choreo support
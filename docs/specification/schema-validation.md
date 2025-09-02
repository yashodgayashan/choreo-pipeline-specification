# Pipeline Schema Validation

## Overview

The Choreo Pipeline Specification provides a JSON Schema for validating pipeline configurations. This schema can be used with various tools to validate your pipeline YAML files before deployment.

## Schema Location

The schema file is located at: `docs/specification/pipeline-schema.json`

## Validation Methods

### Using NPM Package (ajv-cli)

Install the validation tool:
```bash
npm install -g ajv-cli ajv-formats
```

Validate a pipeline file:
```bash
# Validate a single pipeline file
ajv validate -s docs/specification/pipeline-schema.json -d my-pipeline.yaml --strict=false

# Validate multiple pipeline files
ajv validate -s docs/specification/pipeline-schema.json -d "pipelines/*.yaml" --strict=false
```

### Using Python Package (jsonschema)

Install the validation tool:
```bash
pip install jsonschema pyyaml
```

Create a validation script (`validate.py`):
```python
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
        print(f"✅ {pipeline_file} is valid")
        return True
    except ValidationError as e:
        print(f"❌ {pipeline_file} is invalid:")
        print(f"   Error: {e.message}")
        print(f"   Path: {' -> '.join(str(p) for p in e.path)}")
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
```

Run validation:
```bash
python validate.py my-pipeline.yaml docs/specification/pipeline-schema.json
```

### Using Node.js Script

Create a validation script (`validate.js`):
```javascript
const fs = require('fs');
const yaml = require('js-yaml');
const Ajv = require('ajv');

function validatePipeline(pipelineFile, schemaFile) {
  // Load schema
  const schema = JSON.parse(fs.readFileSync(schemaFile, 'utf8'));
  
  // Load pipeline YAML
  const pipeline = yaml.load(fs.readFileSync(pipelineFile, 'utf8'));
  
  // Create validator
  const ajv = new Ajv({ strict: false });
  const validate = ajv.compile(schema);
  
  // Validate
  const valid = validate(pipeline);
  
  if (valid) {
    console.log(`✅ ${pipelineFile} is valid`);
    return true;
  } else {
    console.log(`❌ ${pipelineFile} is invalid:`);
    validate.errors.forEach(err => {
      console.log(`   ${err.instancePath}: ${err.message}`);
    });
    return false;
  }
}

// CLI usage
if (process.argv.length !== 4) {
  console.log('Usage: node validate.js <pipeline.yaml> <schema.json>');
  process.exit(1);
}

const pipelineFile = process.argv[2];
const schemaFile = process.argv[3];

if (validatePipeline(pipelineFile, schemaFile)) {
  process.exit(0);
} else {
  process.exit(1);
}
```

Install dependencies and run:
```bash
npm install js-yaml ajv
node validate.js my-pipeline.yaml docs/specification/pipeline-schema.json
```

## CI/CD Integration

### GitHub Actions

Add validation to your GitHub Actions workflow:
```yaml
name: Validate Pipelines

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install validator
        run: npm install -g ajv-cli ajv-formats
      
      - name: Validate pipelines
        run: |
          for file in pipelines/*.yaml; do
            echo "Validating $file..."
            ajv validate -s docs/specification/pipeline-schema.json -d "$file" --strict=false
          done
```

### Pre-commit Hook

Add validation as a pre-commit hook (`.pre-commit-config.yaml`):
```yaml
repos:
  - repo: local
    hooks:
      - id: validate-pipelines
        name: Validate Pipeline YAML
        entry: ajv validate -s docs/specification/pipeline-schema.json -d
        language: system
        files: \.yaml$
        pass_filenames: true
```

## Schema Features

The schema validates:

### Required Fields
- `steps` array (at least one step required)

### Optional Fields
- `pipelineType`: Must be "build" or "automation"
- `version`: Version string (e.g., "1.0")
- `env`: Environment variables
- `arguments`: Pipeline parameters and artifacts
- `volumes`: Volume configurations
- `resources`: CPU and memory limits
- `templates`: Reusable template definitions

### Step Validation
Each step must have:
- `name`: Step identifier
- One of: `template`, `inlineScript`, or `parallel`

Optional step properties:
- `when`: Conditional execution
- `timeout`: Step timeout
- `retryStrategy`: Retry configuration
- `continueOn`: Error handling
- `env`: Step-specific environment variables
- `arguments`: Step parameters
- `outputs`: Output parameters and artifacts
- `resources`: Resource limits
- `volumeMounts`: Volume mount configurations

### Common Validation Errors

1. **Missing required field**:
   ```
   Error: must have required property 'steps'
   ```
   Solution: Add a `steps` array to your pipeline

2. **Invalid enum value**:
   ```
   Error: /pipelineType must be equal to one of the allowed values: build, automation
   ```
   Solution: Use either "build" or "automation" for pipelineType

3. **Pattern mismatch**:
   ```
   Error: /resources/requests/memory must match pattern "^[0-9]+[KMG]i?$"
   ```
   Solution: Use proper format like "256Mi" or "2Gi"

4. **Step missing required field**:
   ```
   Error: /steps/0 must have required property 'template'
   ```
   Solution: Each step needs either template, inlineScript, or parallel

## VS Code Integration

Add schema validation to VS Code by adding to `.vscode/settings.json`:
```json
{
  "yaml.schemas": {
    "./docs/specification/pipeline-schema.json": ["pipelines/*.yaml", "*.pipeline.yaml"]
  }
}
```

This provides real-time validation and autocompletion in VS Code.

## Updates and Versioning

The schema is versioned along with the specification. When the specification changes:
1. The schema is updated to reflect new fields or constraints
2. The version field in the schema is updated
3. Validation tools automatically use the new schema

## Contributing

To contribute to the schema:
1. Update the schema file at `docs/specification/pipeline-schema.json`
2. Add test cases for new validation rules
3. Update this documentation with any new validation requirements
4. Submit a pull request with your changes
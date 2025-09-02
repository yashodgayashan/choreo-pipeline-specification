# Using LLMs with Choreo Pipeline Specification

## Overview

This guide helps developers leverage Large Language Models (LLMs) like ChatGPT, Claude, or GitHub Copilot to quickly generate and understand Choreo pipeline configurations. LLMs can accelerate your pipeline development by providing instant examples, explanations, and troubleshooting assistance.

## Key Context for LLM Prompts

When prompting LLMs about Choreo pipelines, provide this context for better results:

**Essential Context:**
```
I'm working with Choreo Pipeline Specification, which:
- Uses YAML configuration format
- Is built on Argo Workflows but abstracts Kubernetes complexity
- Provides these environment variables:
  - $REPOSITORY_DIR: where the source code is checked out
  - $WORKSPACE: working directory for pipeline operations
  - $IMAGE_NAME: the name of the built container image
- Has pre-built templates like:
  - choreo/buildpack-build@v1 for building applications
  - choreo/trivy-scan@v1 for security scanning
  - choreo/docker-build@v1 for Docker images
- Supports two pipeline types: 'build' (for CI) and 'automation' (for general tasks)
```

## Quick Start Prompts

### Generate a Basic Build Pipeline

**Enhanced Prompt with Context:**
```
Generate a Choreo pipeline YAML for building a Node.js application.

Context:
- Choreo provides $REPOSITORY_DIR (source code location) and $WORKSPACE (working directory)
- Use choreo/buildpack-build@v1 template for building
- Use choreo/trivy-scan@v1 for security scanning
- The pipeline should:
  1. Install dependencies
  2. Run linting
  3. Run tests with Jest
  4. Build the application
  5. Scan for security vulnerabilities
```

**Expected Output:**
```yaml
steps:
  - name: Install Dependencies
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm install --frozen-lockfile

  - name: Lint Code
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm run lint

  - name: Run Tests
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm test -- --coverage
      cp -r coverage $WORKSPACE/

  - name: Build Application
    template: choreo/buildpack-build@v1
    
  - name: Security Scan
    template: choreo/trivy-scan@v1
    arguments:
      parameters:
        - name: scan_type
          value: "image"
        - name: target
          value: "$IMAGE_NAME"
```

### Convert Existing Pipelines

**Enhanced Prompt with Context:**
```
Convert this GitHub Actions workflow to a Choreo pipeline.

Choreo pipeline context:
- Uses simple YAML with 'steps' array
- Environment: $REPOSITORY_DIR (code location), $WORKSPACE (working dir), $IMAGE_NAME (built image)
- Templates available: choreo/buildpack-build@v1, choreo/docker-build@v1, choreo/trivy-scan@v1
- Scripts use inlineScript with #!/bin/bash
- No need for checkout action (code is already in $REPOSITORY_DIR)

GitHub Actions workflow to convert:
[paste your GitHub Actions YAML]
```

### Explain Pipeline Components

**Prompt:**
```
Explain what this Choreo pipeline step does:
steps:
  - name: parallel-tests
    parallel:
      - template: unit-tests
      - template: integration-tests
    continueOn:
      error: false
```

## Common Use Cases

### 1. Building Multi-Language Projects

**Enhanced Prompt with Context:**
```
Create a Choreo pipeline for a monorepo project.

Context:
- Choreo provides $REPOSITORY_DIR where code is checked out
- Use $WORKSPACE for storing build artifacts
- Can run steps in parallel using parallel syntax
- Project structure:
  - backend/ (Python Flask API)
  - frontend/ (React with TypeScript)
  - shared/ (TypeScript libraries)
  
Requirements:
- Install dependencies for each component
- Run tests for each component
- Build all components (parallel where possible)
- Generate artifacts in $WORKSPACE
```

### 2. Adding Security Scanning

**Enhanced Prompt with Context:**
```
Add comprehensive security scanning to a Choreo pipeline.

Context:
- Choreo provides choreo/trivy-scan@v1 for container scanning
- Code is in $REPOSITORY_DIR, reports go to $WORKSPACE
- Built image name is in $IMAGE_NAME

Add these security checks:
1. Dependency vulnerability scanning (npm audit, pip check)
2. Container image scanning with Trivy
3. SAST (Static Application Security Testing)
4. Secret detection in source code
5. License compliance checking

Generate reports in $WORKSPACE for each scan.
```

### 3. Implementing Testing Strategies

**Enhanced Prompt with Context:**
```
Create a comprehensive testing pipeline in Choreo.

Context:
- Code is in $REPOSITORY_DIR
- Test reports should be saved to $WORKSPACE
- Can run tests in parallel using parallel syntax
- Use continueOn: error: false for non-blocking tests

Implement these test levels:
1. Unit tests with coverage reporting
2. Integration tests with database
3. API contract tests
4. Performance tests
5. Accessibility tests for frontend

All tests should generate reports in $WORKSPACE.
```

## Advanced Prompts

### Resource Optimization

**Prompt:**
```
Optimize this Choreo pipeline for performance:
- Identify steps that can run in parallel
- Suggest appropriate resource limits
- Add caching where beneficial
[paste your pipeline YAML]
```

### Error Handling

**Prompt:**
```
Add comprehensive error handling to my Choreo pipeline:
- Retry flaky tests
- Send notifications on failure
- Create debug artifacts
- Implement graceful degradation
```

### Template Creation

**Prompt:**
```
Create a reusable Choreo template for:
- Running database migrations
- Input: database connection string
- Output: migration status
- Include rollback capability
```

## Troubleshooting with LLMs

### Debug Pipeline Failures

**Prompt:**
```
My Choreo pipeline is failing with this error:
[paste error message]
Here's my pipeline configuration:
[paste relevant YAML]
What's wrong and how do I fix it?
```

### Performance Issues

**Prompt:**
```
My Choreo pipeline takes 45 minutes to complete.
Here's the configuration: [paste YAML]
How can I make it faster?
```

### Configuration Validation

**Prompt:**
```
Review this Choreo pipeline configuration for:
- Security best practices
- Performance optimizations
- Potential issues
[paste your YAML]
```

## Best Practices for LLM Prompts

### 1. Be Specific with Context
Instead of: "Create a pipeline"
Use: "Create a Choreo pipeline for a Java Spring Boot application. Context: Code is in $REPOSITORY_DIR, use choreo/buildpack-build@v1 for building, include Maven tests, checkstyle linting, and choreo/trivy-scan@v1 for security scanning. Save test reports to $WORKSPACE."

### 2. Provide Choreo-Specific Context
Always include:
- Environment variables: $REPOSITORY_DIR, $WORKSPACE, $IMAGE_NAME
- Available templates: choreo/buildpack-build@v1, choreo/trivy-scan@v1, etc.
- Pipeline structure: simple steps array, no pipelineType needed
- Your technology stack and requirements
- Where to save outputs (usually $WORKSPACE)

### 3. Iterate and Refine
Start with a basic pipeline and refine:
1. "Generate a basic Node.js build pipeline"
2. "Add test coverage reporting"
3. "Include dependency caching"
4. "Add conditional deployment"

### 4. Ask for Explanations
**Prompt:**
```
Explain each step of this pipeline and why it's structured this way:
[paste pipeline YAML]
```

## Sample LLM Conversations

### Conversation 1: Building from Scratch

**You:** "I need a Choreo pipeline for my Python Flask API. Context: Code is in $REPOSITORY_DIR, save artifacts to $WORKSPACE"

**LLM:** [Provides pipeline with proper paths]

**You:** "Add PostgreSQL integration tests that run from $REPOSITORY_DIR/tests"

**LLM:** [Updates with database setup and correct paths]

**You:** "Make the tests run in parallel and save all reports to $WORKSPACE"

**LLM:** [Restructures for parallel execution with report saving]

### Conversation 2: Migration Assistance

**You:** "Convert my Jenkins pipeline to Choreo format"

**LLM:** [Provides initial conversion]

**You:** "The Jenkins pipeline uses custom plugins for notifications"

**LLM:** [Suggests Choreo alternatives]

## LLM-Friendly Pipeline Patterns

### Pattern 1: Microservices Build
```yaml
# Tell LLM: "Generate a Choreo pipeline for building multiple microservices
# Context: Services are in $REPOSITORY_DIR/services/*, use $WORKSPACE for artifacts"

steps:
  - name: Detect Changes
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      # Detect which services changed
      git diff --name-only HEAD~1 | grep "^services/" | cut -d/ -f2 | sort -u > $WORKSPACE/changed-services.txt
    
  - name: Build Services
    parallel:
      - name: Build Service A
        inlineScript: |
          #!/bin/bash
          cd $REPOSITORY_DIR/services/service-a
          npm install --frozen-lockfile
          npm run build
          cp -r dist $WORKSPACE/service-a/
      - name: Build Service B
        inlineScript: |
          #!/bin/bash
          cd $REPOSITORY_DIR/services/service-b
          go build -o $WORKSPACE/service-b/app
      - name: Build Service C
        inlineScript: |
          #!/bin/bash
          cd $REPOSITORY_DIR/services/service-c
          mvn clean package
          cp target/*.jar $WORKSPACE/service-c/
```

### Pattern 2: Quality Gate Pattern
```yaml
# Tell LLM: "Create a Choreo pipeline with quality gates
# Context: Use $REPOSITORY_DIR for code, $WORKSPACE for reports, fail if quality gates don't pass"

steps:
  - name: Run Quality Checks
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      # Run linting
      npm run lint > $WORKSPACE/lint-report.txt
      
      # Run tests with coverage
      npm test -- --coverage --json --outputFile=$WORKSPACE/test-report.json
      
      # Check coverage threshold
      coverage=$(cat $WORKSPACE/test-report.json | jq '.coverageMap.total.lines.pct')
      if (( $(echo "$coverage < 80" | bc -l) )); then
        echo "Coverage $coverage% is below 80% threshold"
        exit 1
      fi
    
  - name: Security Gate
    template: choreo/trivy-scan@v1
    arguments:
      parameters:
        - name: scan_type
          value: "fs"
        - name: target
          value: "$REPOSITORY_DIR"
        - name: exit_code
          value: "1"  # Fail on vulnerabilities
```

## Limitations and Considerations

### What LLMs Can Do Well:
- Generate pipeline structure and syntax
- Convert between different CI/CD formats
- Suggest optimizations and best practices
- Explain pipeline behavior
- Troubleshoot common issues

### What to Verify Manually:
- Specific Choreo template versions
- Environment-specific configurations
- Security credentials and secrets
- Resource limits for your infrastructure
- Compliance requirements

## Getting the Most from LLMs

1. **Start Simple**: Begin with basic pipelines and incrementally add complexity
2. **Use Examples**: Provide existing pipelines as reference
3. **Validate Output**: Always test generated configurations in a safe environment
4. **Learn Patterns**: Use LLM explanations to understand Choreo patterns
5. **Build a Library**: Save useful prompts and responses for future reference

## Example Prompt Library

### For New Projects:
```
"Create a production-ready Choreo pipeline for a [technology] application.

Context:
- Source code is in $REPOSITORY_DIR
- Save all artifacts and reports to $WORKSPACE
- Built image name will be in $IMAGE_NAME
- Use choreo/buildpack-build@v1 for building
- Use choreo/trivy-scan@v1 for security scanning

Include:
- Dependency installation from $REPOSITORY_DIR
- Linting and code quality checks
- Unit and integration tests with coverage
- Security vulnerability scanning
- Build artifacts saved to $WORKSPACE
- All reports in $WORKSPACE for CI visibility"
```

### For Optimization:
```
"Analyze this Choreo pipeline and suggest improvements for:
- Build speed
- Resource usage
- Reliability
- Maintainability
[paste pipeline]"
```

### For Debugging:
```
"This Choreo pipeline step is failing:

Context:
- Running in Choreo with $REPOSITORY_DIR=/workspace/source
- $WORKSPACE=/workspace/output
- Error: [error message]

Step configuration:
[paste YAML]

What I've checked:
- File exists in $REPOSITORY_DIR
- Permissions on $WORKSPACE
- [other checks]

What are possible causes and solutions?"
```

## Conclusion

LLMs are powerful tools for accelerating Choreo pipeline development. Use them to:
- Bootstrap new pipelines quickly
- Learn Choreo patterns and best practices
- Troubleshoot issues efficiently
- Convert existing CI/CD configurations
- Optimize pipeline performance

Remember to always validate LLM-generated configurations and adapt them to your specific requirements and security policies.

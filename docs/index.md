# Choreo Pipeline Specification

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/wso2/choreo-pipeline-specification/releases)
[![License](https://img.shields.io/badge/license-Apache%202.0-green.svg)](https://github.com/wso2/choreo-pipeline-specification/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/docs-latest-brightgreen.svg)](specification/overview.md)

The official specification for defining pipelines in the Choreo Internal Developer Platform (IDP). This specification provides a YAML-based configuration system built on top of Argo Workflows, supporting build pipelines, automation workflows, and general-purpose task orchestration while abstracting away Kubernetes complexity.

## Quick Start

### Basic Pipeline

Here's a simple pipeline to get you started. Choreo provides environment variables:
- `$REPOSITORY_DIR` - Where your repository is checked out
- `$WORKSPACE` - Working directory for pipeline operations
- `$IMAGE_NAME` - The name of the built container image

```yaml
steps:
  - name: Build
    template: choreo/buildpack-build@v1
  
  - name: Test
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm test
```

### Pipeline with Linting and Security Scan

For a more comprehensive pipeline with code quality and security checks:

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
      # or for Python: pylint src/
      # or for Java: mvn checkstyle:check
  
  - name: Run Tests
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm test -- --coverage
      # Copy coverage reports to workspace
      cp -r coverage $WORKSPACE/
  
  - name: Build Application
    template: choreo/buildpack-build@v1
  
  - name: Security Scan
    template: choreo/trivy-scan@v1
```

## Documentation

### Core Documentation

- [**Overview**](specification/overview.md) - Introduction to the pipeline specification
- [**Core Concepts**](specification/concepts.md) - Fundamental concepts and architecture
- [**Configuration Guide**](specification/pipeline-configuration.md) - Complete configuration reference
- [**API Reference**](specification/README.md) - Detailed API documentation
- [**LLM Guide**](LLMs.md) - Using AI assistants to generate pipelines

### Examples

Browse our [example pipelines](examples/README.md) organized by pipeline type:

- [**Build Pipelines**](examples/build/hello-world.md) - Start with simple build examples
- [**Automation Pipelines**](examples/automation/scheduled-tasks.md) - Scheduled and long-running tasks
- [**All Examples**](examples/README.md) - Complete examples catalog

## Key Features

### Simple Yet Powerful

- **Declarative YAML** - Easy-to-understand configuration
- **Template Reusability** - Create and share pipeline components
- **Parallel Execution** - Optimize pipeline performance
- **Built-in Templates** - Pre-configured templates for common tasks

### Enterprise Ready

- **Security First** - Secure secrets management
- **Resource Management** - Fine-grained CPU/memory control
- **Retry Strategies** - Automatic error recovery
- **Conditional Execution** - Dynamic pipeline behavior

### Developer Friendly

- **Minimal Learning Curve** - Start simple, grow complex
- **Comprehensive Examples** - Learn by example
- **JSON Schema Validation** - Catch errors early
- **Extensive Documentation** - Everything you need to know

## Repository Structure

```
choreo-pipeline-specification/
├── docs/                      # Documentation
│   ├── specification/         # Core specification and API docs
│   ├── choreo-templates/      # Choreo templates documentation
│   ├── examples/              # Example documentation
│   ├── guides/                # How-to guides and migrations
│   └── LLMs.md                # Guide for using LLMs with pipelines
├── examples/                  # Example pipeline YAML files
│   ├── build/                 # Build pipeline examples
│   └── automation/            # Automation pipeline examples
├── CONTRIBUTING.md            # Contribution guidelines
└── README.md                  # Main repository README
```

## Pipeline Components

### Steps

The basic building blocks of a pipeline:

```yaml
steps:
  - name: my-step
    inlineScript: |
      #!/bin/bash
      echo "Hello, World!"
```

### Templates

Reusable pipeline components:

```yaml
templates:
  - name: test-runner
    inlineScript: |
      #!/bin/bash
      npm test
```

### Environment Variables

Secure configuration management:

```yaml
env:
  - name: API_URL
    value: "{{VARIABLES.API_URL}}"
  - name: API_TOKEN
    value: "{{SECRETS.API_TOKEN}}"
```

### Resource Management

Control resource allocation:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1000m"
```

## Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0.0 | 2025-10 | Initial release |

See [CHANGELOG.md](https://github.com/wso2/choreo-pipeline-specification/blob/main/CHANGELOG.md) for detailed version history.

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](https://github.com/wso2/choreo-pipeline-specification/blob/main/CONTRIBUTING.md) for details on:

- Reporting issues
- Suggesting enhancements
- Submitting pull requests
- Development setup
- Style guidelines

## Validation

Pipeline configurations are automatically validated by the Choreo Console when you save them. The console provides:

- **Schema validation** - Configuration structure is validated against the specification
- **Template validation** - Template references and parameters are verified
- **Secret validation** - Secrets are validated against the specification and whether values are provided
- **Variable validation** - Variables are validated against the specification and whether values are provided

## Status

- Core specification complete
- Basic examples available
- API reference documented
- JSON schema validation
- Advanced examples in progress
- Video tutorials planned

## Getting Help

- Read the [documentation](specification/overview.md)
- Browse [examples](examples/)
- Report [issues](https://github.com/wso2/choreo-pipeline-specification/issues)

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/wso2/choreo-pipeline-specification/blob/main/LICENSE) file for details.

## About WSO2

WSO2 is an open-source technology provider that delivers software to create and manage APIs and cloud-native applications.

---

**[Documentation](specification/overview.md)** | **[Examples](examples/README.md)** | **[API Reference](specification/README.md)** | **[Choreo Templates](choreo-templates/overview.md)** | **[LLM Guide](LLMs.md)**

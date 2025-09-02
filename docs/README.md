# Choreo Pipeline Specification

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/wso2/choreo-pipeline-specification/releases)
[![License](https://img.shields.io/badge/license-Apache%202.0-green.svg)](https://github.com/wso2/choreo-pipeline-specification/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/docs-latest-brightgreen.svg)](specification/overview.md)

The official specification for defining pipelines in the Choreo Internal Developer Platform (IDP). This specification provides a YAML-based configuration system built on top of Argo Workflows, supporting build pipelines, deployment pipelines, automation workflows, and general-purpose task orchestration while abstracting away Kubernetes complexity.

## Quick Start

Following is a sample simple pipeline configuration which can be used in Choreo. Please refer to the [Quick Start Guide](guides/quick-start.md) for more details.

```yaml
steps:
  - name: Build
    template: choreo/buildpack-build@v1
  
  - name: Test
    inlineScript: |
      #!/bin/bash
      npm test
  
  - name: Deploy
    when: "{{VARIABLES.DEPLOYMENT_TRACKS.DEPLOYMENT_BRANCH}} == 'main'"
    template: choreo/deploy@v1
```

**Get started with our [Quick Start Guide](guides/quick-start.md) and [Examples](examples/README.md)**

## Documentation

### Core Documentation

- [**Overview**](specification/overview.md) - Introduction to the pipeline specification
- [**Core Concepts**](specification/concepts.md) - Fundamental concepts and architecture
- [**Configuration Guide**](specification/pipeline-configuration.md) - Complete configuration reference
- [**API Reference**](specification/README.md) - Detailed API documentation
- [**LLM Guide**](LLMs.md) - Using AI assistants to generate pipelines

### Examples

Browse our [example pipelines](examples/) organized by pipeline type:

- [**Build Pipelines**](examples/build/) - Pipelines for building and testing
- [**Deployment Pipelines**](examples/deployment/) - Deployment and release pipelines  
- [**Automation Pipelines**](examples/automation/) - Scheduled and long-running tasks
- [**Choreo Templates**](examples/choreo-templates/) - Template usage examples

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
├── docs/                      # Specification documentation
│   ├── specification/         # Core specification and API docs
│   ├── choreo-templates/      # Choreo templates documentation
│   ├── guides/                # How-to guides and migrations
│   └── LLMs.md                # Guide for using LLMs with pipelines
├── examples/                  # Example pipelines
│   ├── build/                 # Build pipeline examples
│   ├── deployment/            # Deployment pipeline examples
│   ├── automation/            # Automation pipeline examples
│   └── choreo-templates/      # Template usage examples
├── CONTRIBUTING.md            # Contribution guidelines
└── README.md                  # This file
```

## Installation & Usage

### For Pipeline Developers

1. Open the Choreo Console
2. Navigate to your component settings
3. Configure your pipeline either by:
   - **Plain Text**: Paste your YAML configuration directly
   - **Repository**: Provide the repository location containing your pipeline YAML
4. Save and deploy to trigger pipeline execution

### For Documentation Integration

See [INTEGRATION.md](INTEGRATION.md) for detailed instructions on integrating with docs-choreo-dev.

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
| 1.0.0 | 2024-01 | Initial release |

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details on:

- Reporting issues
- Suggesting enhancements
- Submitting pull requests
- Development setup
- Style guidelines

## Validation

Pipeline configurations are automatically validated by the Choreo Console when you save them. The console provides:

- **Real-time syntax validation** - YAML syntax errors are highlighted immediately
- **Schema validation** - Configuration structure is validated against the specification
- **Template validation** - Template references and parameters are verified
- **Error reporting** - Clear error messages with line numbers and suggestions

## Integration with docs-choreo-dev

This specification is integrated with the main Choreo documentation. See:

- [Integration Guide](https://github.com/wso2/choreo-pipeline-specification/blob/main/INTEGRATION.md) - How to integrate with MkDocs
- [Main Documentation](https://github.com/wso2/docs-choreo-dev) - Choreo documentation repository

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

WSO2 is an open-source technology provider that delivers software to create, deploy, and manage APIs and cloud-native applications.

---

**[Documentation](specification/overview.md)** | **[Examples](examples/README.md)** | **[API Reference](api-reference/index.md)** | **[Choreo Templates](choreo-templates/overview.md)** | **[Contributing](https://github.com/wso2/choreo-pipeline-specification/blob/main/CONTRIBUTING.md)**

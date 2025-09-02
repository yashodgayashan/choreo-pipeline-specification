# Choreo Pipeline Specification

## Overview

The Choreo Pipeline Specification defines a YAML-based configuration system for creating customizable pipelines for various purposes including build pipelines, automation workflows, and general-purpose task orchestration. Built on top of Argo Workflows, it provides a simplified, user-friendly interface while abstracting away Kubernetes complexity.

## Supported Pipeline Types

The specification supports two distinct pipeline types, each with specific characteristics and constraints. See [Pipeline Types Specification](./pipeline-types.md) for detailed information.

### 1. Build Pipelines
Build pipelines optimized for fast feedback on code changes:

- **Purpose**: Build, test, and validate code
- **Constraints**: 60-minute default timeout, User can configure the timeout and resource limits
- **Features**: Automatic source checkout, artifact management
- **Triggers**: Code merge requests if configured, otherwise manual
- **Use Cases**: Code compilation, unit testing, security scanning

### 2. Automation Pipelines
General-purpose automation with no constraints:

- **Purpose**: Any automation or orchestration task
- **Constraints**: None - unlimited resources and duration as configured by the user
- **Features**: persistent storage, external integrations
- **Triggers**: Manual execution
- **Use Cases**: Data processing, ETL, infrastructure automation

## Key Features

- **Declarative Configuration**: Define pipelines using simple YAML syntax
- **Multi-Purpose**: Support for build and automation workflows
- **Template Reusability**: Create and share reusable pipeline components
- **Parallel Execution**: Support for parallel step execution
- **Security First**: Built-in secrets management and security scanning
- **Resource Management**: Fine-grained control over CPU and memory resources
- **Retry Strategies**: Configurable retry policies for resilient pipelines
- **Conditional Logic**: Dynamic pipeline behavior based on conditions
- **Data Persistence**: Volume management for stateful workflows

## Getting Started

To use this specification, configure your pipeline in the Choreo Console either by pasting the YAML directly or pointing to a repository containing your pipeline configuration. For detailed examples and step-by-step instructions, see the [Quick Start Guide](../guides/quick-start.md) and [Examples](../../examples/).

## Architecture

The pipeline specification is designed with the following principles:

1. **Simplicity**: Easy to understand and write
2. **Flexibility**: Support for various use cases
3. **Security**: Secure by default with proper secrets handling
4. **Compatibility**: Works with existing Choreo infrastructure
5. **Extensibility**: Easy to add new features and templates

## Components

The specification consists of several key components:

- **Steps**: Individual tasks in the pipeline
- **Templates**: Reusable units of work
- **Container Templates**: Container configurations
- **Volume Claims**: Persistent storage configurations
- **Environment Variables**: Configuration and secrets management

## Configuration Management

Pipeline configurations are managed through the Choreo Console:

- **Console UI**: Direct YAML editing in the web interface
- **Repository Integration**: Point to YAML files in your git repository

## Next Steps

- [Core Concepts](./concepts.md) - Understand the fundamental concepts
- [Pipeline Configuration](./pipeline-configuration.md) - Learn how to configure pipelines  
- [API Reference](./README.md) - Detailed API documentation
- [Examples](../../examples/README.md) - Browse example configurations
- [Choreo Templates](../choreo-templates/overview.md) - Pre-built template reference
- [Quick Start Guide](../guides/quick-start.md) - Get started quickly
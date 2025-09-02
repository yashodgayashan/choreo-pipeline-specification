# API Reference

## Overview

This section provides a complete reference for all configuration options available in the Choreo Pipeline Specification. The specification supports multiple pipeline types including build pipelines, automation workflows, and general-purpose task orchestration.

## Schema Validation

A JSON Schema is provided at [`pipeline-schema.json`](./pipeline-schema.json) for validating your pipeline configurations. See the [Schema Validation Guide](./schema-validation.md) for details on using npm or Python packages to validate your pipelines.

## Table of Contents

- [Pipeline Structure](./pipeline-structure.md)
- [Steps](./steps.md)
- [Templates](./templates.md)
- [Environment Variables](./environment-variables.md)
- [Resources](./resources.md)
- [Choreo Templates](./built-in-templates.md)
- [Schema Validation](./schema-validation.md) - JSON Schema for validating pipelines

## Quick Reference

### Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `steps` | Array | Yes | Pipeline steps to execute |
| `templates` | Array | No | Custom template definitions |
| `containerTemplates` | Array | No | Container template definitions |
| `arguments` | Object | No | Global pipeline arguments |
| `volumeClaimTemplates` | Array | No | Persistent volume claims |

### Step Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Step name (alphanumeric) |
| `template` | String | No* | Template reference |
| `inlineScript` | String | No* | Inline bash script |
| `containerSet` | Object | No* | Container set configuration |
| `env` | Array | No | Environment variables |
| `volumeMounts` | Array | No | Volume mount points |
| `resources` | Object | No | Resource limits |
| `retryStrategy` | Object | No | Retry configuration |
| `timeout` | String | No | Step timeout |
| `when` | String | No | Conditional execution |
| `arguments` | Object | No | Step arguments |
| `continueOn` | Object | No | Error handling |

*One of `template`, `inlineScript`, or `containerSet` is required

### Template Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Template name |
| `inlineScript` | String | No* | Bash script (Choreo) |
| `script` | Object | No* | Script config (Argo) |
| `containerSet` | Object | No* | Container set (Argo) |
| `image` | String | No | Container image |
| `env` | Array | No | Environment variables |
| `volumeMounts` | Array | No | Volume mounts |
| `resources` | Object | No | Resource limits |
| `inputs` | Object | No | Input parameters |
| `outputs` | Object | No | Output parameters |

*One of `inlineScript`, `script`, or `containerSet` is required

## Environment Variable Reference

### Variable Types

**Note**: Organization, Project, Component, and Data Plane level variables and secrets are only available in CI pipelines.

| Type | Syntax | Example |
|------|--------|---------|
| Component Variables | `{{VARIABLES.NAME}}` | `{{VARIABLES.API_URL}}` |
| Component Secrets | `{{SECRETS.NAME}}` | `{{SECRETS.API_TOKEN}}` |
| Organization Variables | `{{VARIABLES.ORG.NAME}}` | `{{VARIABLES.ORG.REGISTRY_URL}}` |
| Organization Secrets | `{{SECRETS.ORG.NAME}}` | `{{SECRETS.ORG.API_KEY}}` |
| Project Variables | `{{VARIABLES.PROJECT.NAME}}` | `{{VARIABLES.PROJECT.DATABASE_URL}}` |
| Project Secrets | `{{SECRETS.PROJECT.NAME}}` | `{{SECRETS.PROJECT.DB_PASSWORD}}` |
| Component Variables (Explicit) | `{{VARIABLES.COMPONENT.NAME}}` | `{{VARIABLES.COMPONENT.PORT}}` |
| Component Secrets (Explicit) | `{{SECRETS.COMPONENT.NAME}}` | `{{SECRETS.COMPONENT.SECRET_KEY}}` |
| Data Plane Variables | `{{VARIABLES.DT.NAME}}` | `{{VARIABLES.DT.ENDPOINT}}` |
| Data Plane Secrets | `{{SECRETS.DT.NAME}}` | `{{SECRETS.DT.ACCESS_TOKEN}}` |
| Inline | Direct value | `"production"` |
| System | `$VARIABLE` | `$REPOSITORY_DIR` |

### System Variables

| Variable | Description | Availability |
|----------|-------------| ------------|
| `$REPOSITORY_DIR` | Repository root directory | CI pipelines |
| `$WORKSPACE` | Working directory for pipeline operations | All pipelines |
| `$IMAGE_NAME` | The name of the built container image | CI pipelines |

## Template Expressions

### Expression Types

| Type | Syntax | Example |
|------|--------|---------|
| Workflow | `{{workflow.*}}` | `{{workflow.uid}}` |
| Steps | `{{steps.*}}` | `{{steps.build.outputs.parameters.id}}` |
| Arguments | `{{arguments.*}}` | `{{arguments.parameters.version}}` |
| Inputs | `{{inputs.*}}` | `{{inputs.parameters.env}}` |

## Resource Units

### Memory

| Unit | Description | Example |
|------|-------------|---------|
| `Ki` | Kibibytes | `512Ki` |
| `Mi` | Mebibytes | `256Mi` |
| `Gi` | Gibibytes | `4Gi` |

### CPU

| Unit | Description | Example |
|------|-------------|---------|
| `m` | Millicores | `500m` |
| (none) | Cores | `2` |

## Retry Policies

| Policy | Description |
|--------|-------------|
| `Always` | Retry on any failure |
| `OnFailure` | Retry on task failure |
| `OnError` | Retry on system error |

## Volume Access Modes

| Mode | Description |
|------|-------------|
| `ReadWriteOnce` | Read-write by single node |
| `ReadOnlyMany` | Read-only by multiple nodes |
| `ReadWriteMany` | Read-write by multiple nodes |

## Choreo Templates

Choreo provides pre-built templates for common pipeline tasks. For complete documentation, see [Choreo Templates](./built-in-templates.md).

### Quick Reference

| Category | Templates | Documentation |
|----------|-----------|---------------|
| **Build** | buildpack-build, docker-build, npm-build, maven-build, gradle-build, go-build, python-build | [Build Templates](../choreo-templates/build-templates.md) |
| **Security** | trivy-scan, checkov-scan, sast-scan, dependency-scan, secret-scan, dockerfile-scan, license-scan | [Security Templates](../choreo-templates/security-templates.md) |

## Error Handling

### Continue On Options

```yaml
continueOn:
  error: true     # Continue on error
  failed: false   # Stop on failure
```

## Conditional Execution

### Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equals | `"{{var}} == 'value'"` |
| `!=` | Not equals | `"{{var}} != 'value'"` |
| `&&` | AND | `"{{a}} && {{b}}"` |
| `\|\|` | OR | `"{{a}} \|\| {{b}}"` |

## Limitations

- Only script and containerSet templates supported
- Single level of parallelism
- No DAG templates
- No HTTP templates
- No recursive templates

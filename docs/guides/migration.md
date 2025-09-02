# Migration Guide Overview

## Introduction

This guide provides an overview of migrating from various CI platforms to Choreo Pipeline Specification. Detailed platform-specific migration guides are available for each major CI system.

## Supported Platforms

We provide comprehensive migration guides for the following platforms:

### [GitHub Actions](./migrations/github-actions.md)
Convert GitHub Actions workflows to Choreo pipelines, covering:

- Workflow structure translation
- Actions marketplace replacements
- Matrix strategy migration
- Reusable workflows and composite actions

### [GitLab CI](./migrations/gitlab-ci.md)
Migrate GitLab CI configurations to Choreo, including:

- `.gitlab-ci.yml` conversion
- GitLab services to containerSet
- Dynamic child pipelines
- Review apps and environments

### [Bitbucket Pipelines](./migrations/bitbucket.md)
Convert Bitbucket Pipelines to Choreo, covering:

- `bitbucket-pipelines.yml` migration
- Pipes to Choreo templates mapping
- Deployment environments
- Custom pipelines with variables

## Quick Comparison

### Pipeline File Locations

| Platform | Configuration File | Choreo Equivalent |
|----------|-------------------|-------------------|
| GitHub Actions | `.github/workflows/*.yml` | Choreo Console YAML Configuration |
| GitLab CI | `.gitlab-ci.yml` | Choreo Console YAML Configuration |
| Bitbucket | `bitbucket-pipelines.yml` | Choreo Console YAML Configuration |

### Core Concepts Mapping

| Concept | GitHub Actions | GitLab CI | Bitbucket | Choreo |
|---------|---------------|-----------|-----------|--------|
| **Pipeline Unit** | Job | Job | Step | Step |
| **Parallel Execution** | `matrix` | `parallel` | `parallel` | `- -` syntax |
| **Conditionals** | `if:` | `only/except` | `condition` | `when:` |
| **Variables** | `env:` | `variables:` | `variables:` | `env:` |
| **Secrets** | Secrets | CI Variables | Repository Variables | `{{SECRETS}}` |
| **Artifacts** | `upload-artifact` | `artifacts:` | `artifacts:` | Not supported yet (planned) |
| **Caching** | `actions/cache` | `cache:` | `caches:` | `volumeClaimTemplates` |

## Choosing the Right Pipeline Type

When migrating to Choreo, select the appropriate pipeline type based on your use case:

### CI Pipelines
**Use for:** Build, test, and validation workflows

- Automatic source code checkout
- Artifact management (not supported yet, planned for future)
- Caching via `volumeClaimTemplates`
- 60-minute default timeout limit 

**Migrate from:**
- GitHub Actions workflow jobs
- GitLab CI build/test stages
- Bitbucket default pipelines

### Automation Pipelines
**Use for:** Long-running tasks, scheduled jobs, data processing
- 60-minute default timeout limit 
- Organization level
- Persistent storage

**Migrate from:**
- GitHub Actions workflow jobs
- GitLab CI pipelines
- Bitbucket pipelines

## Current Limitations & Workarounds

### Artifacts (Not Yet Supported)
**Current Status**: Artifact management is not currently supported but is planned for future releases.

**Migration Impact**:

- Pipelines using artifact upload/download between jobs need alternative approaches
- Multi-stage pipelines sharing build outputs require workarounds

**Recommended Workarounds**:

1. **Persistent Volumes**: Use `volumeClaimTemplates` to share files between steps
2. **External Storage**: Upload to S3, GCS, or other cloud storage via scripts
3. **Container Registries**: Store build artifacts as container images
4. **Single Pipeline**: Combine multiple jobs into one pipeline where possible

### Caching (Supported via Volume Claim Templates)
**Current Status**: Fully supported using Kubernetes persistent volumes

**Migration Path**:

- Replace cache actions/configurations with `volumeClaimTemplates`
- Implement cache logic in build scripts
- Use persistent volumes for dependency caching (node_modules, .m2, etc.)

## Common Migration Patterns

### Environment Variables
```yaml
# Choreo approach for all platforms
env:
  - name: NODE_ENV
    value: "production"
  - name: API_KEY
    value: "{{SECRETS.API_KEY}}"
  - name: VERSION
    value: "{{VARIABLES.VERSION}}"
```

### Conditional Execution
```yaml
# Choreo unified syntax
steps:
  - name: Deploy to Production
    when: "{{CI_BRANCH}} == 'main'"
    template: choreo/deploy@v1
```

### Parallel Execution
```yaml
# Choreo parallel steps
steps:
  - - name: Test Frontend
      template: test-frontend
    - name: Test Backend
      template: test-backend
    - name: Test API
      template: test-api
```

### Caching with Volume Claim Templates
```yaml
# Persistent cache storage for build dependencies
volumeClaimTemplates:
  - metadata:
      name: node-cache
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "5Gi"

steps:
  - name: Install Dependencies
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR

      # Use cached node_modules if available
      if [ -d "/cache/node_modules" ]; then
        cp -r /cache/node_modules ./
      fi

      npm ci

      # Update cache
      cp -r node_modules /cache/
    volumeMounts:
      - name: node-cache
        mountPath: /cache
```

### Artifact Management (Future Capability)
```yaml
# Artifact management is not currently supported in Choreo
# This capability is planned for future releases
#
# For now, use alternative approaches:
# 1. Store build outputs in persistent volumes
# 2. Upload to external storage (S3, etc.) via scripts
# 3. Use container registries for image artifacts

# Example workaround using persistent storage
volumeClaimTemplates:
  - metadata:
      name: build-artifacts
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "10Gi"

steps:
  - name: Build Application
    inlineScript: |
      #!/bin/bash
      cd $REPOSITORY_DIR
      npm run build

      # Store build artifacts in persistent volume
      cp -r dist/ /artifacts/
    volumeMounts:
      - name: build-artifacts
        mountPath: /artifacts
```

## Quick Start Examples

### Simple Build Pipeline
```yaml
steps:
  - name: Build and Test
    container:
      image: node:18
    inlineScript: |
      #!/bin/bash
      npm ci
      npm test
      npm run build
```

## Getting Help

### Resources
- **[Examples](../../examples/)** - Sample pipelines for common scenarios
- **[Choreo Templates](../choreo-templates/overview.md)** - Pre-built templates reference
- **[API Reference](../specification/README.md)** - Complete syntax documentation
- **[Best Practices](./best-practices.md)** - Recommended patterns and practices

### Platform-Specific Guides
- **[GitHub Actions Migration](./migrations/github-actions.md)** - Complete GitHub Actions conversion
- **[GitLab CI Migration](./migrations/gitlab-ci.md)** - GitLab CI migration guide
- **[Bitbucket Migration](./migrations/bitbucket.md)** - Bitbucket Pipelines conversion

### Support
For complex migrations or specific questions:

1. Review the platform-specific migration guide
2. Check the [Troubleshooting Guide](./troubleshooting.md)
3. Consult the [Choreo Templates Reference](../choreo-templates/overview.md)
4. Contact Choreo support for assistance

## Next Steps

1. **Choose your platform** - Select the migration guide for your current CI system
2. **Start small** - Begin with a simple pipeline to familiarize yourself with Choreo
3. **Use templates** - Leverage Choreo's built-in templates to simplify migration
4. **Iterate and improve** - Optimize your pipelines after initial migration
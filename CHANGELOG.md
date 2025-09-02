# Changelog

All notable changes to the Choreo Pipeline Specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-01-15

### Added
- Initial release of Choreo Pipeline Specification
- Support for three pipeline types: CI, CD, and Automation
- Comprehensive template system with built-in Choreo templates
- YAML-based pipeline configuration
- Complete API reference documentation
- Example pipelines for all supported use cases
- JSON schema validation for pipeline configurations
- Migration guides for Jenkins, GitHub Actions, GitLab CI, and Bitbucket
- Integration support with MkDocs documentation system

### Pipeline Types
- **CI Pipelines**: 60-minute timeout, 8Gi memory, 4 CPU limits
- **CD Pipelines**: 120-minute timeout, environment-aware deployments
- **Automation Pipelines**: Unlimited resources and duration

### Built-in Templates
- **Build Templates**: buildpack-build, docker-build, npm-build, maven-build, gradle-build, go-build, python-build
- **Security Templates**: trivy-scan, checkov-scan, sast-scan, dependency-scan, secret-scan, dockerfile-scan, license-scan
- **Testing Templates**: test-runner, integration-test, performance-test, e2e-test
- **Deployment Templates**: deploy, kubectl-apply, helm-deploy, blue-green-deploy, canary-deploy, rollback
- **Notification Templates**: slack-notify, email-notify, teams-notify
- **Utility Templates**: git-clone, cache-restore, cache-save, artifact-upload, artifact-download

### Documentation
- Complete specification documentation
- API reference for all components
- Migration guides from major CI/CD platforms
- Best practices and troubleshooting guides
- Comprehensive examples organized by pipeline type

### Features
- Declarative YAML configuration
- Parallel step execution
- Template reusability and composition
- Environment variable management with Variables and Secrets
- Resource management and retry strategies
- Conditional execution with when expressions
- Volume management for persistent data
- Artifact management system

## [Unreleased]

### Planned
- Video tutorials and documentation
- Additional built-in templates
- Enhanced integration examples
- Performance optimizations
- Extended validation features

---

For more detailed information about each release, please refer to the [releases page](https://github.com/wso2/choreo-pipeline-specification/releases).
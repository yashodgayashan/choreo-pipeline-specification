# Security Templates

Security templates provide comprehensive security scanning capabilities for applications, containers, and infrastructure.

## Available Security Templates

### `choreo/trivy-scan@v1`
Container and filesystem vulnerability scanning with comprehensive security checks.

**Usage:**
```yaml
- name: Vulnerability Scan
  template: choreo/trivy-scan@v1
```

### `choreo/checkov-scan@v1`
Infrastructure as Code security scanning for multiple frameworks.

**Usage:**
```yaml
- name: IaC Security Scan
  template: choreo/checkov-scan@v1
```

## Template Features

### Trivy Scan Features

- **Multi-format Scanning**: Supports containers, filesystems, and repositories
- **Comprehensive Vulnerability Database**: Up-to-date CVE database
- **Secret Detection**: Built-in secret scanning capabilities
- **Configuration Scanning**: Detects security misconfigurations
- **Multiple Output Formats**: SARIF, JSON, table formats

### Checkov Scan Features

- **Multi-framework Support**: Terraform, CloudFormation, Kubernetes, Dockerfile
- **Policy as Code**: Comprehensive security policy checks
- **Custom Rules**: Support for custom security policies
- **Compliance Frameworks**: CIS, NIST, PCI-DSS compliance checks
- **Integration Ready**: SARIF output for security dashboards

## Best Practices

### Security Pipeline Implementation
```yaml
steps:
  # Parallel security scanning
  - - name: Container Security Scan
      template: choreo/trivy-scan@v1

    - name: Infrastructure Security Scan
      template: choreo/checkov-scan@v1
```

### Error Handling
Add retry strategies for security operations:
```yaml
steps:
  - name: Security Scan
    template: choreo/trivy-scan@v1
    retryStrategy:
      limit: 2
      retryPolicy: OnFailure
    timeout: "10m"
```

### Security Gates
Use conditional execution for security gates:
```yaml
steps:
  - name: Security Scan
    template: choreo/trivy-scan@v1

  - name: Deploy
    when: "{{steps.security-scan.outputs.result}} == 'passed'"
    template: deploy-application
```

## Security Scanning Strategy

### Multi-Layer Security
Implement security scanning at multiple levels:

- **Container Security**: Use `choreo/trivy-scan@v1` for container vulnerability scanning
- **Infrastructure Security**: Use `choreo/checkov-scan@v1` for Infrastructure as Code scanning
- **Comprehensive Coverage**: Combine both templates for complete security coverage

### Integration with Build Process
```yaml
steps:
  - name: Build Application
    template: choreo/docker-build@v1

  - name: Security Scan
    template: choreo/trivy-scan@v1

  - name: Infrastructure Scan
    template: choreo/checkov-scan@v1
```

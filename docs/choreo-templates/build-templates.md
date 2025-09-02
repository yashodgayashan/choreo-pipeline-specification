# Build Templates

Build templates provide standardized ways to build applications across different languages and frameworks.

## Available Build Templates

### `choreo/buildpack-build@v1`
Build applications using Cloud Native Buildpacks with automatic language detection.

**Usage:**
```yaml
- name: Build with Buildpacks
  template: choreo/buildpack-build@v1
```

### `choreo/docker-build@v1`
Build Docker images with multi-platform support and advanced caching.

**Usage:**
```yaml
- name: Build Docker Image
  template: choreo/docker-build@v1
```

## Template Features

### Buildpack Build Features

- **Automatic Language Detection**: Detects application language and framework
- **Optimized Images**: Creates minimal, secure container images
- **Dependency Caching**: Caches dependencies for faster builds
- **Multi-language Support**: Supports Node.js, Java, Python, Go, and more

### Docker Build Features

- **Multi-platform Support**: Build for multiple architectures
- **Layer Caching**: Efficient Docker layer caching
- **Security Scanning**: Built-in vulnerability scanning
- **Build Optimization**: Optimized build process for faster execution

## Best Practices

### Choose the Right Template
- Use `choreo/buildpack-build@v1` for standard applications that don't require custom Dockerfiles
- Use `choreo/docker-build@v1` when you need custom containerization or specific Docker configurations

### Template Usage
```yaml
steps:
  - name: Build Application
    template: choreo/buildpack-build@v1

  - name: Build Custom Image
    template: choreo/docker-build@v1
```

### Error Handling
Add retry strategies for build operations:
```yaml
steps:
  - name: Build
    template: choreo/docker-build@v1
    retryStrategy:
      limit: 2
      retryPolicy: OnFailure
    timeout: "15m"
```

# Go Build Pipeline

High-performance Go application pipeline demonstrating parallel execution, container sets, comprehensive testing, and cross-platform builds.

## Pipeline Configuration

```yaml
# Go Build Pipeline
# Demonstrates parallel execution, container sets, and Go-specific build patterns
# Use Case: Building and testing Go applications with comprehensive quality checks

steps:
  # Step 1: Setup and dependency management
  - name: Setup Dependencies
    inlineScript: |
      #!/bin/bash
      echo "Setting up Go environment..."
      cd $REPOSITORY_DIR
      go version
      go mod download
      go mod verify
      echo "Dependencies downloaded and verified"

  # Step 2: Parallel quality checks using container sets
  - - name: Unit Tests
      containerSet:
        - name: test-runner
          image: golang:1.21-alpine
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
      inlineScript: |
        #!/bin/bash
        echo "Running Go unit tests..."
        cd $REPOSITORY_DIR
        go test -v -race -coverprofile=coverage.out ./...
        go tool cover -func=coverage.out
        echo "Unit tests completed"
      timeout: 300
      retries: 2

    - name: Integration Tests
      containerSet:
        - name: integration-test
          image: golang:1.21-alpine
          resources:
            requests:
              memory: "1Gi"
              cpu: "1000m"
            limits:
              memory: "2Gi"
              cpu: "2000m"
      inlineScript: |
        #!/bin/bash
        echo "Running integration tests..."
        cd $REPOSITORY_DIR
        go test -v -tags=integration ./...
        echo "Integration tests completed"
      timeout: 600
      retries: 1
      env:
        - name: DATABASE_URL
          value: "{{SECRETS.TEST_DATABASE_URL}}"
        - name: REDIS_URL
          value: "{{VARIABLES.TEST_REDIS_URL}}"

    - name: Code Quality
      containerSet:
        - name: linter
          image: golangci/golangci-lint:v1.54-alpine
          resources:
            requests:
              memory: "256Mi"
              cpu: "500m"
      inlineScript: |
        #!/bin/bash
        echo "Running code quality checks..."
        cd $REPOSITORY_DIR
        golangci-lint run --timeout 5m
        echo "Code quality checks completed"
      timeout: 300
      retries: 1

  # Step 3: Build application
  - name: Build Application
    containerSet:
      - name: builder
        image: golang:1.21-alpine
        resources:
          requests:
            memory: "1Gi"
            cpu: "1000m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
    inlineScript: |
      #!/bin/bash
      echo "Building Go application..."
      cd $REPOSITORY_DIR

      # Build for multiple platforms
      GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o bin/app-linux-amd64 ./cmd/app
      GOOS=darwin GOARCH=amd64 go build -ldflags="-w -s" -o bin/app-darwin-amd64 ./cmd/app

      echo "Application built successfully for multiple platforms"
      ls -la bin/
    timeout: 300
    retries: 1
    env:
      - name: CGO_ENABLED
        value: "0"
      - name: GOPROXY
        value: "{{VARIABLES.GO_PROXY}}"

  # Step 4: Security scanning
  - name: Security Scan
    template: choreo/trivy-scan@v1
    timeout: 300
    retries: 1

  # Step 5: Build summary
  - name: Build Summary
    inlineScript: |
      #!/bin/bash
      echo "=== Go Build Summary ==="
      echo "Application: {{VARIABLES.APP_NAME}}"
      echo "Version: {{VARIABLES.VERSION}}"
      echo "Go Version: $(go version)"
      echo "Build completed successfully"
      echo "Artifacts ready for deployment"
    env:
      - name: BUILD_NUMBER
        value: "{{VARIABLES.BUILD_NUMBER}}"
```

## Key Features

### **Container Sets**
- **Custom Resource Allocation**: Different containers optimized for specific tasks
- **Isolated Environments**: Each step runs in purpose-built containers
- **Performance Tuning**: Resource requests and limits tailored to workload

### **Parallel Execution**
```yaml
# Parallel quality checks
- - name: Unit Tests
    # High-performance test runner
  - name: Integration Tests
    # Database-connected testing
  - name: Code Quality
    # Static analysis and linting
```

### **Timeouts & Retries**
```yaml
timeout: 300        # 5 minute timeout
retries: 2          # Retry failed steps twice
```

### **Cross-Platform Builds**
```bash
# Build for multiple architectures
GOOS=linux GOARCH=amd64 go build -o bin/app-linux-amd64
GOOS=darwin GOARCH=amd64 go build -o bin/app-darwin-amd64
```

## Go-Specific Optimizations

### **Build Performance**
- **Module Caching**: `go mod download` with verification
- **Compiler Optimizations**: `-ldflags="-w -s"` for smaller binaries
- **CGO Disabled**: `CGO_ENABLED=0` for static linking
- **Proxy Configuration**: Configurable `GOPROXY` for dependency management

### **Testing Strategy**
- **Race Detection**: `-race` flag for concurrent code testing
- **Coverage Analysis**: Detailed coverage reporting
- **Integration Testing**: Separate test suite with database connectivity
- **Build Tags**: `-tags=integration` for test isolation

### **Quality Gates**
- **golangci-lint**: Comprehensive static analysis
- **Security Scanning**: Container vulnerability assessment
- **Module Security**: Dependency verification and audit

## Configuration

### **Variables**
```yaml
VARIABLES:
  APP_NAME: "go-microservice"
  VERSION: "1.0.0"
  GO_PROXY: "https://proxy.golang.org"
  TEST_REDIS_URL: "redis://localhost:6379"
  BUILD_NUMBER: "123"
```

### **Secrets**
```yaml
SECRETS:
  TEST_DATABASE_URL: "postgresql://username:password@hostname:5432/test_database"
```

## Project Structure

```
/
├── cmd/
│   └── app/
│       └── main.go
├── internal/
│   ├── handlers/
│   ├── models/
│   └── services/
├── pkg/
│   └── utils/
├── go.mod
├── go.sum
└── .golangci.yml
```

**Perfect for**: Go microservices, CLI tools, and high-performance applications requiring comprehensive testing and cross-platform deployment.
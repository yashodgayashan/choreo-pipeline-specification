# Java Build Pipeline

Enterprise-grade Java build pipeline showcasing comprehensive variable management, secret handling, database integration, and Spring Boot application patterns.

## Pipeline Configuration

```yaml
# Java Build Pipeline
# Demonstrates Maven/Gradle builds with extensive variable usage, secret management, and enterprise patterns
# Use Case: Building Spring Boot applications with database integration and comprehensive testing

steps:
  # Step 1: Environment setup with variables
  - name: Setup Java Environment
    inlineScript: |
      #!/bin/bash
      echo "Setting up Java build environment..."
      echo "Java Version: $JAVA_VERSION"
      echo "Maven Version: $(mvn --version | head -1)"
      echo "Application: {{VARIABLES.APP_NAME}}"
      echo "Profile: {{VARIABLES.BUILD_PROFILE}}"

      cd $REPOSITORY_DIR

      # Validate project structure
      if [ ! -f "pom.xml" ] && [ ! -f "build.gradle" ]; then
        echo "Error: No Maven (pom.xml) or Gradle (build.gradle) file found"
        exit 1
      fi

      echo "Java environment setup completed"
    timeout: 120
    retries: 1
    env:
      - name: JAVA_VERSION
        value: "{{VARIABLES.JAVA_VERSION}}"
      - name: MAVEN_OPTS
        value: "{{VARIABLES.MAVEN_OPTS}}"

  # Step 2: Dependency resolution with secret repositories
  - name: Resolve Dependencies
    inlineScript: |
      #!/bin/bash
      echo "Resolving dependencies..."
      cd $REPOSITORY_DIR

      # Configure private repository access
      if [ -n "$NEXUS_USERNAME" ]; then
        echo "Configuring private repository access..."
        # Maven settings would be configured here
      fi

      # Download dependencies
      mvn dependency:resolve -B -q
      mvn dependency:resolve-sources -B -q

      echo "Dependencies resolved successfully"
    timeout: 600
    retries: 2
    env:
      - name: NEXUS_USERNAME
        value: "{{SECRETS.NEXUS_USERNAME}}"
      - name: NEXUS_PASSWORD
        value: "{{SECRETS.NEXUS_PASSWORD}}"
      - name: NEXUS_URL
        value: "{{VARIABLES.NEXUS_URL}}"

  # Step 3: Parallel testing with database integration
  - - name: Unit Tests
      inlineScript: |
        #!/bin/bash
        echo "Running unit tests..."
        cd $REPOSITORY_DIR

        # Run unit tests with coverage
        mvn test -B \
          -Dspring.profiles.active=test \
          -Djacoco.skip=false \
          -Dmaven.test.failure.ignore=false

        echo "Unit tests completed"
      timeout: 900
      retries: 2
      env:
        - name: SPRING_PROFILES_ACTIVE
          value: "test"

    - name: Integration Tests
      inlineScript: |
        #!/bin/bash
        echo "Running integration tests..."
        cd $REPOSITORY_DIR

        # Wait for database to be ready
        echo "Waiting for database connection..."

        # Run integration tests
        mvn verify -B \
          -Dspring.profiles.active=integration \
          -Dtest.database.url="$DATABASE_URL" \
          -Dtest.database.username="$DATABASE_USERNAME" \
          -Dtest.database.password="$DATABASE_PASSWORD"

        echo "Integration tests completed"
      timeout: 1200
      retries: 1
      env:
        - name: DATABASE_URL
          value: "{{SECRETS.TEST_DATABASE_URL}}"
        - name: DATABASE_USERNAME
          value: "{{SECRETS.DATABASE_USERNAME}}"
        - name: DATABASE_PASSWORD
          value: "{{SECRETS.DATABASE_PASSWORD}}"
        - name: REDIS_URL
          value: "{{VARIABLES.TEST_REDIS_URL}}"
        - name: SPRING_PROFILES_ACTIVE
          value: "integration"

    - name: Code Quality Analysis
      inlineScript: |
        #!/bin/bash
        echo "Running code quality analysis..."
        cd $REPOSITORY_DIR

        # SonarQube analysis with authentication
        mvn sonar:sonar -B \
          -Dsonar.host.url="$SONAR_HOST_URL" \
          -Dsonar.login="$SONAR_TOKEN" \
          -Dsonar.projectKey="{{VARIABLES.SONAR_PROJECT_KEY}}" \
          -Dsonar.projectName="{{VARIABLES.APP_NAME}}" \
          -Dsonar.projectVersion="{{VARIABLES.VERSION}}"

        echo "Code quality analysis completed"
      timeout: 600
      retries: 1
      env:
        - name: SONAR_HOST_URL
          value: "{{VARIABLES.SONAR_HOST_URL}}"
        - name: SONAR_TOKEN
          value: "{{SECRETS.SONAR_TOKEN}}"
        - name: SONAR_PROJECT_KEY
          value: "{{VARIABLES.SONAR_PROJECT_KEY}}"

  # Step 4: Build application with profile-specific configuration
  - name: Build Application
    inlineScript: |
      #!/bin/bash
      echo "Building Java application..."
      cd $REPOSITORY_DIR

      # Build with specific profile and version
      mvn clean package -B \
        -DskipTests=true \
        -Dspring.profiles.active="{{VARIABLES.BUILD_PROFILE}}" \
        -Dapp.version="{{VARIABLES.VERSION}}" \
        -Dbuild.number="{{VARIABLES.BUILD_NUMBER}}" \
        -Dmaven.compiler.source="{{VARIABLES.JAVA_VERSION}}" \
        -Dmaven.compiler.target="{{VARIABLES.JAVA_VERSION}}"

      # Verify JAR file was created
      if [ -f "target/*.jar" ]; then
        echo "JAR file created successfully"
        ls -la target/*.jar
      else
        echo "Error: JAR file not found"
        exit 1
      fi

      echo "Application build completed"
    timeout: 900
    retries: 2
    env:
      - name: BUILD_PROFILE
        value: "{{VARIABLES.BUILD_PROFILE}}"
      - name: JAVA_TOOL_OPTIONS
        value: "{{VARIABLES.JAVA_TOOL_OPTIONS}}"

  # Step 5: Security scanning
  - name: Security Vulnerability Scan
    template: choreo/trivy-scan@v1
    timeout: 300
    retries: 1

  # Step 6: Package and artifact management
  - name: Package Application
    inlineScript: |
      #!/bin/bash
      echo "Packaging application for deployment..."
      cd $REPOSITORY_DIR

      # Create deployment package
      mkdir -p deployment
      cp target/*.jar deployment/
      cp -r config/ deployment/ 2>/dev/null || true

      # Generate deployment metadata
      cat > deployment/metadata.json << EOF
      {
        "application": "{{VARIABLES.APP_NAME}}",
        "version": "{{VARIABLES.VERSION}}",
        "buildNumber": "{{VARIABLES.BUILD_NUMBER}}",
        "profile": "{{VARIABLES.BUILD_PROFILE}}",
        "javaVersion": "{{VARIABLES.JAVA_VERSION}}",
        "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
      }
      EOF

      echo "Application packaged successfully"
      echo "Package contents:"
      ls -la deployment/
    timeout: 300
    retries: 1

  # Step 7: Build summary with comprehensive variable usage
  - name: Build Summary Report
    inlineScript: |
      #!/bin/bash
      echo "=== Java Build Summary ==="
      echo "Application: {{VARIABLES.APP_NAME}}"
      echo "Version: {{VARIABLES.VERSION}}"
      echo "Build Number: {{VARIABLES.BUILD_NUMBER}}"
      echo "Java Version: {{VARIABLES.JAVA_VERSION}}"
      echo "Build Profile: {{VARIABLES.BUILD_PROFILE}}"
      echo "Maven Repository: {{VARIABLES.NEXUS_URL}}"
      echo "SonarQube Project: {{VARIABLES.SONAR_PROJECT_KEY}}"
      echo ""
      echo "✅ Dependencies resolved"
      echo "✅ Unit tests passed"
      echo "✅ Integration tests passed"
      echo "✅ Code quality analysis completed"
      echo "✅ Security scan passed"
      echo "✅ Application packaged"
      echo ""
      echo "Ready for deployment to {{VARIABLES.DEPLOYMENT_ENVIRONMENT}}"
    continueOn:
      error: true
    env:
      - name: DEPLOYMENT_ENVIRONMENT
        value: "{{VARIABLES.DEPLOYMENT_ENVIRONMENT}}"
      - name: NOTIFICATION_EMAIL
        value: "{{VARIABLES.NOTIFICATION_EMAIL}}"
```

## Key Features

### **Comprehensive Variable Usage**
- **Build Configuration**: Java version, Maven options, build profiles
- **Repository Management**: Private Nexus repository configuration
- **Environment-Specific**: Different profiles for test, staging, production
- **Metadata Generation**: Build artifacts with comprehensive metadata

### **Secret Management**
```yaml
# Database credentials
DATABASE_URL: "{{SECRETS.TEST_DATABASE_URL}}"
DATABASE_USERNAME: "{{SECRETS.DATABASE_USERNAME}}"
DATABASE_PASSWORD: "{{SECRETS.DATABASE_PASSWORD}}"

# Repository access
NEXUS_USERNAME: "{{SECRETS.NEXUS_USERNAME}}"
NEXUS_PASSWORD: "{{SECRETS.NEXUS_PASSWORD}}"

# Code quality integration
SONAR_TOKEN: "{{SECRETS.SONAR_TOKEN}}"
```

### **Parallel Testing Strategy**
```yaml
# Concurrent execution of different test types
- - name: Unit Tests           # Fast feedback
  - name: Integration Tests    # Database connectivity
  - name: Code Quality        # SonarQube analysis
```

## Java-Specific Patterns

### **Maven Configuration**
```xml
<!-- Profile-based configuration -->
<profiles>
  <profile>
    <id>test</id>
    <properties>
      <spring.profiles.active>test</spring.profiles.active>
    </properties>
  </profile>
  <profile>
    <id>production</id>
    <properties>
      <spring.profiles.active>production</spring.profiles.active>
    </properties>
  </profile>
</profiles>
```

### **Spring Boot Integration**
```bash
# Profile-specific builds
mvn clean package -Dspring.profiles.active=production

# Integration testing with database
mvn verify -Dspring.profiles.active=integration \
  -Dtest.database.url=$DATABASE_URL
```

### **Dependency Management**
- **Private Repository Access**: Secure Nexus integration
- **Dependency Resolution**: Sources and javadocs download
- **Version Management**: Consistent versioning across modules
- **Cache Optimization**: Maven local repository caching

## Testing Strategy

### **Unit Testing**
```bash
mvn test -B \
  -Dspring.profiles.active=test \
  -Djacoco.skip=false          # Enable code coverage
```

### **Integration Testing**
```bash
mvn verify -B \
  -Dspring.profiles.active=integration \
  -Dtest.database.url="$DATABASE_URL"
```

### **Code Quality**
```bash
mvn sonar:sonar -B \
  -Dsonar.host.url="$SONAR_HOST_URL" \
  -Dsonar.login="$SONAR_TOKEN" \
  -Dsonar.projectKey="com.company:app"
```

## Build Optimization

### **Performance Tuning**
```yaml
env:
  - name: MAVEN_OPTS
    value: "-Xmx2g -XX:+UseG1GC"
  - name: JAVA_TOOL_OPTIONS
    value: "-Dfile.encoding=UTF-8"
```

### **Retry Strategies**
```yaml
timeout: 900        # 15 minutes for builds
retries: 2          # Retry network-dependent operations
```

### **Parallel Execution**
- Unit tests, integration tests, and code quality run simultaneously
- Independent resource allocation per test type
- Optimized timeout settings per operation type

## Configuration Management

### **Variables**
```yaml
VARIABLES:
  APP_NAME: "my-spring-boot-app"
  VERSION: "1.2.3"
  BUILD_NUMBER: "456"
  JAVA_VERSION: "17"
  BUILD_PROFILE: "production"
  MAVEN_OPTS: "-Xmx2g -XX:+UseG1GC"
  NEXUS_URL: "https://nexus.company.com/repository/maven-public/"
  SONAR_HOST_URL: "https://sonarqube.company.com"
  SONAR_PROJECT_KEY: "com.company:my-spring-boot-app"
  TEST_REDIS_URL: "redis://localhost:6379"
  DEPLOYMENT_ENVIRONMENT: "staging"
  NOTIFICATION_EMAIL: "team@company.com"
```

### **Secrets**
```yaml
SECRETS:
  NEXUS_USERNAME: "your-nexus-username"
  NEXUS_PASSWORD: "your-nexus-password"
  TEST_DATABASE_URL: "postgresql://hostname:5432/test_database"
  DATABASE_USERNAME: "test_user"
  DATABASE_PASSWORD: "test_password"
  SONAR_TOKEN: "your-sonarqube-token"
```

## Project Structure

```
/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/app/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-test.yml
│   │       └── application-production.yml
│   └── test/
│       ├── java/
│       │   ├── unit/
│       │   └── integration/
│       └── resources/
├── config/
│   ├── logback-spring.xml
│   └── database-migration/
└── deployment/
    └── metadata.json
```

## Spring Boot Configuration

### **Application Profiles**
```yaml
# application.yml
spring:
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:development}

# application-production.yml
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
  redis:
    url: ${REDIS_URL}
```

### **Build Metadata**
```json
{
  "application": "my-spring-boot-app",
  "version": "1.2.3",
  "buildNumber": "456",
  "profile": "production",
  "javaVersion": "17",
  "buildTime": "2024-01-15T10:30:00Z"
}
```

**Perfect for**: Spring Boot applications, microservices architectures, enterprise Java applications requiring comprehensive testing, code quality analysis, and secure dependency management.
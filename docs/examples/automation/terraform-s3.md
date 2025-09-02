# Terraform S3 Automation Pipeline

Simple infrastructure automation pipeline using Terraform to create and manage AWS S3 buckets with proper state management and validation.

## Pipeline Configuration

```yaml
# Terraform S3 Automation Pipeline
# Demonstrates infrastructure automation using Terraform to create AWS S3 buckets
# Use Case: Automated infrastructure provisioning and management

steps:
  # Step 1: Setup Terraform environment
  - name: Setup Terraform
    inlineScript: |
      #!/bin/bash
      echo "Setting up Terraform environment..."

      # Verify Terraform installation
      terraform version

      # Initialize working directory
      cd $REPOSITORY_DIR/terraform

      # Configure AWS credentials
      echo "Configuring AWS credentials..."
      echo "AWS Region: {{VARIABLES.AWS_REGION}}"

      echo "Terraform setup completed"
    timeout: 300
    retries: 1
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"
      - name: AWS_DEFAULT_REGION
        value: "{{VARIABLES.AWS_REGION}}"

  # Step 2: Terraform initialization and validation
  - name: Terraform Init and Validate
    inlineScript: |
      #!/bin/bash
      echo "Initializing Terraform..."
      cd $REPOSITORY_DIR/terraform

      # Initialize Terraform
      terraform init -backend-config="bucket={{VARIABLES.TERRAFORM_STATE_BUCKET}}"

      # Validate configuration
      terraform validate

      # Format check
      terraform fmt -check=true

      echo "Terraform initialization and validation completed"
    timeout: 600
    retries: 2
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"

  # Step 3: Terraform plan
  - name: Terraform Plan
    inlineScript: |
      #!/bin/bash
      echo "Creating Terraform execution plan..."
      cd $REPOSITORY_DIR/terraform

      # Create execution plan
      terraform plan \
        -var="bucket_name={{VARIABLES.S3_BUCKET_NAME}}" \
        -var="environment={{VARIABLES.ENVIRONMENT}}" \
        -var="project={{VARIABLES.PROJECT_NAME}}" \
        -out=tfplan

      echo "Terraform plan created successfully"
    timeout: 300
    retries: 1
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"

  # Step 4: Apply Terraform changes
  - name: Terraform Apply
    inlineScript: |
      #!/bin/bash
      echo "Applying Terraform changes..."
      cd $REPOSITORY_DIR/terraform

      # Apply the planned changes
      terraform apply -auto-approve tfplan

      # Show outputs
      echo "=== Terraform Outputs ==="
      terraform output

      echo "Infrastructure provisioning completed"
    timeout: 900
    retries: 1
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"

  # Step 5: Verify S3 bucket creation
  - name: Verify S3 Resources
    inlineScript: |
      #!/bin/bash
      echo "Verifying S3 bucket creation..."

      # Check if bucket exists
      aws s3 ls s3://{{VARIABLES.S3_BUCKET_NAME}} || {
        echo "Error: S3 bucket not found"
        exit 1
      }

      # List bucket properties
      echo "Bucket created successfully:"
      echo "Name: {{VARIABLES.S3_BUCKET_NAME}}"
      echo "Region: {{VARIABLES.AWS_REGION}}"
      echo "Environment: {{VARIABLES.ENVIRONMENT}}"

      echo "S3 verification completed"
    timeout: 180
    retries: 2
    env:
      - name: AWS_ACCESS_KEY_ID
        value: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
      - name: AWS_SECRET_ACCESS_KEY
        value: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"

  # Step 6: Infrastructure summary
  - name: Infrastructure Summary
    inlineScript: |
      #!/bin/bash
      echo "=== Infrastructure Summary ==="
      echo "Project: {{VARIABLES.PROJECT_NAME}}"
      echo "Environment: {{VARIABLES.ENVIRONMENT}}"
      echo "AWS Region: {{VARIABLES.AWS_REGION}}"
      echo "S3 Bucket: {{VARIABLES.S3_BUCKET_NAME}}"
      echo ""
      echo "✅ Terraform initialized"
      echo "✅ Configuration validated"
      echo "✅ Infrastructure planned"
      echo "✅ Resources provisioned"
      echo "✅ S3 bucket verified"
      echo ""
      echo "Infrastructure automation completed successfully"
    continueOn:
      error: true
```

## Key Features

### **Infrastructure as Code**
- **Terraform State Management**: Remote state storage in S3
- **Configuration Validation**: Syntax and format checking
- **Execution Planning**: Preview changes before applying
- **Resource Verification**: Confirm infrastructure creation

### **AWS Integration**
```bash
# Secure credential management
AWS_ACCESS_KEY_ID: "{{SECRETS.AWS_ACCESS_KEY_ID}}"
AWS_SECRET_ACCESS_KEY: "{{SECRETS.AWS_SECRET_ACCESS_KEY}}"

# Environment-specific configuration
terraform plan \
  -var="bucket_name={{VARIABLES.S3_BUCKET_NAME}}" \
  -var="environment={{VARIABLES.ENVIRONMENT}}"
```

### **Error Handling & Retries**
```yaml
timeout: 600        # 10 minutes for init
retries: 2          # Retry network operations
```

## Terraform Configuration

### **Required Files Structure**
```
terraform/
├── main.tf         # S3 bucket resource definition
├── variables.tf    # Variable declarations
├── outputs.tf      # Output definitions
└── backend.tf      # State backend configuration
```

### **Example main.tf**
```hcl
variable "bucket_name" {
  description = "Name of the S3 bucket"
  type        = string
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string
}

variable "project" {
  description = "Project name"
  type        = string
}

resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "Terraform"
  }
}

resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  bucket = aws_s3_bucket.main.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

output "bucket_name" {
  value = aws_s3_bucket.main.bucket
}

output "bucket_arn" {
  value = aws_s3_bucket.main.arn
}
```

### **Example backend.tf**
```hcl
terraform {
  backend "s3" {
    key            = "terraform.tfstate"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

## Pipeline Workflow

### **1. Environment Setup**
- Verify Terraform installation
- Configure AWS credentials
- Set working directory

### **2. Initialization & Validation**
- Initialize Terraform with remote backend
- Validate configuration syntax
- Check code formatting

### **3. Planning**
- Generate execution plan
- Review proposed changes
- Save plan file for apply step

### **4. Apply Changes**
- Execute the planned infrastructure changes
- Display Terraform outputs
- Confirm resource creation

### **5. Verification**
- Verify S3 bucket exists using AWS CLI
- Confirm bucket properties
- Validate infrastructure state

## Configuration

### **Variables**
```yaml
VARIABLES:
  PROJECT_NAME: "my-project"
  ENVIRONMENT: "development"
  AWS_REGION: "us-east-1"
  S3_BUCKET_NAME: "my-project-dev-bucket-12345"
  TERRAFORM_STATE_BUCKET: "terraform-state-bucket"
```

### **Secrets**
```yaml
SECRETS:
  AWS_ACCESS_KEY_ID: "your-aws-access-key-id"
  AWS_SECRET_ACCESS_KEY: "your-aws-secret-access-key"
```

## Security Best Practices

### **AWS Credentials**
- Store credentials as secrets, never in code
- Use IAM roles with minimal required permissions
- Enable MFA for AWS accounts

### **S3 Security**
- Enable bucket versioning
- Configure server-side encryption
- Implement proper bucket policies
- Enable access logging

### **Terraform State**
- Store state in encrypted S3 bucket
- Use DynamoDB for state locking
- Restrict access to state files

**Perfect for**: Infrastructure automation, cloud resource provisioning, environment setup, and infrastructure-as-code workflows requiring AWS S3 storage.
---
title: Linux User Management Implementation

---

# AWS IAM User & Group Management with Terraform

## 🎯 Project Overview
This project implements a complete AWS IAM infrastructure using Terraform, creating and managing:
- 3 IAM Groups (Developers, Operations, DevOps)
- 7 IAM Users with appropriate group memberships
- AWS Managed Policies for each group
- Console access with secure password management

## 📁 Project Structure
```plaintext
aws-iam-project/
├── main.tf           # Main configuration file
├── variables.tf      # Variable declarations
├── terraform.tfvars  # Variable values
└── outputs.tf        # Output definitions
```

## 💻 Complete Code with Explanation

### 📌 terraform.tfvars
```hcl
# AWS region where resources will be created
region = "us-east-1"

# AWS access credentials
access_key = "AccessKey"         
secret_key = "SecretKey"

# Lists of users for each group
developer_users = ["meraj", "shailendra", "john"]
operations_users = ["farah", "diksha"]
devops_users = ["sanskar" , "kunal"]

# Environment tag value
environment = "production"
```
This file contains the actual values for variables defined in variables.tf.

### 📌 variables.tf
```hcl
# Variable declarations for AWS credentials
variable "access_key" {
  description = "Access key to AWS console"           
}

variable "secret_key" {
  description = "Secret key to AWS console"           
}

variable "region" {
  description = "Region of AWS VPC"
}

# Variables for user lists with default values
variable "developer_users" {
  description = "List of developer usernames"
  type        = list(string)
  default     = ["dev1", "dev2", "dev3"]  # Default values if not specified in terraform.tfvars
}

variable "operations_users" {
  description = "List of operations usernames"
  type        = list(string)
  default     = ["ops1", "ops2"]
}

variable "devops_users" {
  description = "List of devops usernames"
  type        = list(string)
  default     = ["do1", "do2"]
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}
```
This file defines all variables used in the project. Each variable has a description and can have a default value.

### 📌 main.tf
```hcl
# Configure AWS provider with credentials
provider "aws" {
  region     = var.region       # Use region from variables
  access_key = var.access_key   # Use access key from variables
  secret_key = var.secret_key   # Use secret key from variables
}

# Create three IAM groups
resource "aws_iam_group" "developers" {
  name = "developers"    # Create developers group
}

resource "aws_iam_group" "operations" {
  name = "operations"    # Create operations group
}

resource "aws_iam_group" "devops" {
  name = "devops"        # Create devops group
}

# Create IAM users for developers
resource "aws_iam_user" "dev_users" {
  count = length(var.developer_users)    # Create as many users as in the developer_users list
  name  = var.developer_users[count.index]    # Use names from the list
  
  tags = {
    Department = "Development"    # Tag for organization
    Environment = var.environment    # Use environment from variables
  }
}

# Create IAM users for operations
resource "aws_iam_user" "ops_users" {
  count = length(var.operations_users)    # Create users based on operations_users list
  name  = var.operations_users[count.index]
  
  tags = {
    Department = "Operations"
    Environment = var.environment
  }
}

# Create IAM users for devops
resource "aws_iam_user" "do_users" {
  count = length(var.devops_users)    # Create users based on devops_users list
  name  = var.devops_users[count.index]
  
  tags = {
    Department = "DevOps"
    Environment = var.environment
  }
}

# Assign developers to developers group
resource "aws_iam_user_group_membership" "dev_group_membership" {
  count = length(var.developer_users)    # For each developer
  user  = aws_iam_user.dev_users[count.index].name    # Reference the user name
  groups = [aws_iam_group.developers.name]    # Add to developers group
}

# Assign operations users to operations group
resource "aws_iam_user_group_membership" "ops_group_membership" {
  count = length(var.operations_users)
  user  = aws_iam_user.ops_users[count.index].name
  groups = [aws_iam_group.operations.name]
}

# Assign devops users to devops group
resource "aws_iam_user_group_membership" "do_group_membership" {
  count = length(var.devops_users)
  user  = aws_iam_user.do_users[count.index].name
  groups = [aws_iam_group.devops.name]
}

# Attach AWS managed policy to developers group
resource "aws_iam_group_policy_attachment" "developer_policy" {
  group      = aws_iam_group.developers.name
  policy_arn = "arn:aws:iam::aws:policy/AWSCodeCommitPowerUser"    # Gives CodeCommit access
}

# Attach AWS managed policy to operations group
resource "aws_iam_group_policy_attachment" "operations_policy" {
  group      = aws_iam_group.operations.name
  policy_arn = "arn:aws:iam::aws:policy/job-function/SystemAdministrator"    # System admin access
}

# Attach AWS managed policy to devops group
resource "aws_iam_group_policy_attachment" "devops_policy" {
  group      = aws_iam_group.devops.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonDevOpsGuruConsoleFullAccess"    # DevOps tools access
}

# Create login profiles for developers (console access)
resource "aws_iam_user_login_profile" "dev_user_login" {
  count = length(var.developer_users)
  user  = aws_iam_user.dev_users[count.index].name
  password_reset_required = true    # Force password change on first login
}

# Create login profiles for operations users
resource "aws_iam_user_login_profile" "ops_user_login" {
  count = length(var.operations_users)
  user  = aws_iam_user.ops_users[count.index].name
  password_reset_required = true
}

# Create login profiles for devops users
resource "aws_iam_user_login_profile" "do_user_login" {
  count = length(var.devops_users)
  user  = aws_iam_user.do_users[count.index].name
  password_reset_required = true
}
```

### 📌 outputs.tf
```hcl
# Output the list of developer usernames
output "developer_users" {
  value = aws_iam_user.dev_users[*].name
}

# Output the list of operations usernames
output "operations_users" {
  value = aws_iam_user.ops_users[*].name
}

# Output the list of devops usernames
output "devops_users" {
  value = aws_iam_user.do_users[*].name
}

# Output encrypted passwords for developers (sensitive data)
output "developer_passwords" {
  value     = aws_iam_user_login_profile.dev_user_login[*].encrypted_password
  sensitive = true    # Mark as sensitive to hide in logs
}

# Output encrypted passwords for operations users
output "operations_passwords" {
  value     = aws_iam_user_login_profile.ops_user_login[*].encrypted_password
  sensitive = true
}

# Output encrypted passwords for devops users
output "devops_passwords" {
  value     = aws_iam_user_login_profile.do_user_login[*].encrypted_password
  sensitive = true
}
```

## 🔍 Implementation Details

### Group Configurations
1. **Developers Group**
   - Policy: AWSCodeCommitPowerUser
   - Members: meraj, shailendra, john
   - Access: Code repository management

2. **Operations Group**
   - Policy: SystemAdministrator
   - Members: farah, diksha
   - Access: System administration capabilities

3. **DevOps Group**
   - Policy: AmazonDevOpsGuruConsoleFullAccess
   - Members: sanskar, kunal
   - Access: DevOps tools and services

### Project Results
As shown in the screenshots:
1. Terraform successfully created 27 resources
2. Three IAM groups were created (developers, operations, devops)
3. Seven IAM users were created and assigned to their groups
4. All users have console access enabled
5. Passwords were generated for all users (marked as sensitive in outputs)

To deploy this project:
1. Save all files in the same directory
2. Run `terraform init` to initialize
3. Run `terraform plan` to preview changes
4. Run `terraform apply` to create resources

## 📊 Deployment Results

### Resources Created
- ✅ 3 IAM Groups
- ✅ 7 IAM Users
- ✅ Group Policy Attachments
- ✅ User Login Profiles


## 🛠 Maintenance Guide

### Adding New Users
1. Update the appropriate user list in `terraform.tfvars`:
```hcl
developer_users = ["meraj", "shailendra", "john", "new_dev"]
```
2. Run:
```bash
terraform plan
terraform apply
```

### Modifying Policies
1. Update the policy ARN in the policy attachment resource
2. Apply changes:
```bash
terraform plan
terraform apply
```

### Removing Users
1. Remove user from the list in `terraform.tfvars`
2. Run:
```bash
terraform plan
terraform apply
```

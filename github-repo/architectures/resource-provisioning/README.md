# Automated Resource Provisioning Architecture

> Terraform + Lambda Validator + Azure Pipeline Integration

## 🎯 Overview

End-to-end automated infrastructure provisioning system with approval workflows, validation gates, and real-time status updates. Integrates JIRA for ticket management, Teams for approvals, Terraform for IaC, and Lambda for post-deployment validation.

## 🏗️ Architecture Diagram

![Resource Provisioning Flow](../../diagrams/flow-diagrams/resource-provisioning-flow.png)

## 📊 High-Level Flow

```
┌──────────────┐
│ User Request │
│ (Teams Bot)  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ JIRA Ticket  │
│ Auto-Created │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Admin        │
│ Approval Gate│ ◄──── Teams Adaptive Card
└──────┬───────┘
       │ Approved
       ▼
┌──────────────┐
│ Azure        │
│ Pipeline     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Terraform    │
│ Apply        │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Lambda       │
│ Validator    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Status Update│
│ to Teams     │
└──────────────┘
```

## 🔧 Components

### 1. Teams Bot Interface
**Purpose:** User initiates provisioning request

**Command Format:**
```
/provision ec2 --type t3.medium --region us-west-2 --env prod
```

**Bot Response:**
```json
{
  "type": "message",
  "text": "✅ Request received. Creating JIRA ticket...",
  "attachments": [{
    "contentType": "application/vnd.microsoft.card.adaptive",
    "content": {
      "type": "AdaptiveCard",
      "body": [
        {"type": "TextBlock", "text": "Request Details"},
        {"type": "FactSet", "facts": [...]}
      ]
    }
  }]
}
```

### 2. JIRA Integration
**Purpose:** Track provisioning requests

**Auto-Created Ticket:**
```yaml
Project: INFRA
Type: Task
Summary: "Provision EC2 Instance - t3.medium - us-west-2"
Description: |
  Resource Type: EC2 Instance
  Instance Type: t3.medium
  Region: us-west-2
  Environment: prod
  Requested By: user@example.com
  Request Date: 2026-06-22T10:30:00Z
Labels:
  - automation
  - terraform
  - ec2
Priority: Medium
Status: Pending Approval
```

**API Call:**
```python
def create_jira_ticket(resource_details):
    url = f"https://{JIRA_DOMAIN}/rest/api/3/issue"
    payload = {
        "fields": {
            "project": {"key": "INFRA"},
            "summary": f"Provision {resource_details['type']}",
            "description": format_description(resource_details),
            "issuetype": {"name": "Task"},
            "labels": ["automation", "terraform", resource_details['type']]
        }
    }
    response = requests.post(url, json=payload, auth=JIRA_AUTH)
    return response.json()['key']
```

### 3. Admin Approval Gate
**Purpose:** Manual approval before provisioning

**Teams Approval Card:**
```json
{
  "type": "AdaptiveCard",
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.4",
  "body": [
    {
      "type": "Container",
      "items": [
        {
          "type": "TextBlock",
          "text": "🔐 Approval Required",
          "weight": "bolder",
          "size": "large",
          "color": "attention"
        },
        {
          "type": "TextBlock",
          "text": "New infrastructure provisioning request",
          "wrap": true
        },
        {
          "type": "FactSet",
          "facts": [
            {"title": "JIRA Ticket", "value": "INFRA-1234"},
            {"title": "Resource", "value": "EC2 Instance"},
            {"title": "Instance Type", "value": "t3.medium"},
            {"title": "Region", "value": "us-west-2"},
            {"title": "Environment", "value": "prod"},
            {"title": "Estimated Cost", "value": "$30/month"},
            {"title": "Requested By", "value": "user@example.com"}
          ]
        }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.Http",
      "title": "✅ Approve",
      "method": "POST",
      "url": "https://api.example.com/approve",
      "body": "{\"ticket\": \"INFRA-1234\", \"action\": \"approve\"}"
    },
    {
      "type": "Action.Http",
      "title": "❌ Reject",
      "method": "POST",
      "url": "https://api.example.com/approve",
      "body": "{\"ticket\": \"INFRA-1234\", \"action\": \"reject\"}"
    },
    {
      "type": "Action.OpenUrl",
      "title": "View JIRA",
      "url": "https://jira.example.com/browse/INFRA-1234"
    }
  ]
}
```

### 4. Azure Pipeline
**Purpose:** Execute Terraform and orchestrate workflow

**azure-pipelines.yml:**
```yaml
trigger: none

pr: none

parameters:
  - name: jiraTicket
    displayName: 'JIRA Ticket ID'
    type: string
  - name: resourceType
    displayName: 'Resource Type'
    type: string
    values:
      - ec2
      - rds
      - s3
      - vpc
  - name: environment
    displayName: 'Environment'
    type: string
    values:
      - dev
      - staging
      - prod

variables:
  - group: terraform-vars
  - name: TF_VERSION
    value: '1.5.0'

stages:
  - stage: Validate
    displayName: 'Validate Request'
    jobs:
      - job: CheckApproval
        steps:
          - task: Bash@3
            name: VerifyJiraApproval
            inputs:
              targetType: 'inline'
              script: |
                # Check JIRA ticket status
                STATUS=$(curl -u $JIRA_USER:$JIRA_TOKEN \
                  "https://jira.example.com/rest/api/3/issue/${{parameters.jiraTicket}}" \
                  | jq -r '.fields.status.name')
                
                if [ "$STATUS" != "Approved" ]; then
                  echo "##vso[task.complete result=Failed;]Ticket not approved"
                  exit 1
                fi
                
                echo "✅ Approval verified"

  - stage: Plan
    displayName: 'Terraform Plan'
    dependsOn: Validate
    jobs:
      - job: TerraformPlan
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: $(TF_VERSION)
          
          - task: Bash@3
            name: InitTerraform
            inputs:
              targetType: 'inline'
              script: |
                cd terraform/${{parameters.resourceType}}
                terraform init \
                  -backend-config="bucket=terraform-state-bucket" \
                  -backend-config="key=${{parameters.environment}}/${{parameters.resourceType}}.tfstate"
          
          - task: Bash@3
            name: PlanTerraform
            inputs:
              targetType: 'inline'
              script: |
                cd terraform/${{parameters.resourceType}}
                terraform plan \
                  -var-file="environments/${{parameters.environment}}.tfvars" \
                  -out=tfplan
                
                # Save plan summary
                terraform show -json tfplan > plan.json
          
          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: 'terraform/${{parameters.resourceType}}/plan.json'
              artifact: 'terraform-plan'

  - stage: Apply
    displayName: 'Provision Resources'
    dependsOn: Plan
    condition: succeeded()
    jobs:
      - deployment: ApplyTerraform
        environment: ${{parameters.environment}}
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifact: 'terraform-plan'
                
                - task: Bash@3
                  name: ApplyTerraform
                  inputs:
                    targetType: 'inline'
                    script: |
                      cd terraform/${{parameters.resourceType}}
                      terraform apply -auto-approve tfplan
                      
                      # Extract outputs
                      terraform output -json > outputs.json
                      
                      # Store outputs for validator
                      echo "##vso[task.setvariable variable=TerraformOutputs;isOutput=true]$(cat outputs.json)"
                
                - task: PublishPipelineArtifact@1
                  inputs:
                    targetPath: 'terraform/${{parameters.resourceType}}/outputs.json'
                    artifact: 'terraform-outputs'

  - stage: Validate_Deployment
    displayName: 'Validate Deployment'
    dependsOn: Apply
    jobs:
      - job: RunValidator
        steps:
          - task: DownloadPipelineArtifact@2
            inputs:
              artifact: 'terraform-outputs'
          
          - task: AWSShellScript@1
            inputs:
              awsCredentials: 'aws-service-connection'
              regionName: 'us-west-2'
              scriptType: 'inline'
              inlineScript: |
                # Invoke Lambda validator
                aws lambda invoke \
                  --function-name InfrastructureValidator \
                  --payload file://outputs.json \
                  --region us-west-2 \
                  response.json
                
                # Check validation result
                VALIDATION_STATUS=$(jq -r '.validationStatus' response.json)
                
                if [ "$VALIDATION_STATUS" != "PASSED" ]; then
                  echo "❌ Validation failed"
                  jq '.validationErrors' response.json
                  exit 1
                fi
                
                echo "✅ Validation passed"

  - stage: Notify
    displayName: 'Update Status'
    dependsOn: Validate_Deployment
    condition: always()
    jobs:
      - job: NotifyTeams
        steps:
          - task: Bash@3
            inputs:
              targetType: 'inline'
              script: |
                # Determine status
                if [ "$(Agent.JobStatus)" == "Succeeded" ]; then
                  STATUS="✅ Success"
                  COLOR="good"
                else
                  STATUS="❌ Failed"
                  COLOR="attention"
                fi
                
                # Send Teams notification
                curl -H "Content-Type: application/json" \
                  -d "{
                    \"type\": \"message\",
                    \"attachments\": [{
                      \"contentType\": \"application/vnd.microsoft.card.adaptive\",
                      \"content\": {
                        \"type\": \"AdaptiveCard\",
                        \"body\": [{
                          \"type\": \"TextBlock\",
                          \"text\": \"$STATUS: ${{parameters.jiraTicket}}\",
                          \"weight\": \"bolder\",
                          \"color\": \"$COLOR\"
                        }]
                      }
                    }]
                  }" \
                  $(TEAMS_WEBHOOK_URL)
                
                # Update JIRA ticket
                curl -X PUT \
                  -u $JIRA_USER:$JIRA_TOKEN \
                  -H "Content-Type: application/json" \
                  "https://jira.example.com/rest/api/3/issue/${{parameters.jiraTicket}}" \
                  -d "{
                    \"fields\": {
                      \"status\": {\"name\": \"Done\"}
                    }
                  }"
```

### 5. Terraform Modules
**Purpose:** Infrastructure as Code

**Example: EC2 Instance**
```hcl
# terraform/ec2/main.tf

terraform {
  required_version = ">= 1.5.0"
  
  backend "s3" {
    bucket         = "terraform-state-bucket"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
  
  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = var.environment
      JiraTicket  = var.jira_ticket
      CreatedBy   = var.created_by
    }
  }
}

# Security Group
resource "aws_security_group" "instance" {
  name_prefix = "${var.environment}-${var.instance_name}-"
  description = "Security group for ${var.instance_name}"
  vpc_id      = var.vpc_id
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_ssh_cidrs
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

# EC2 Instance
resource "aws_instance" "main" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  
  vpc_security_group_ids = [aws_security_group.instance.id]
  
  iam_instance_profile = aws_iam_instance_profile.instance.name
  
  user_data = templatefile("${path.module}/user_data.sh", {
    environment = var.environment
  })
  
  root_block_device {
    volume_size           = var.root_volume_size
    volume_type           = "gp3"
    encrypted             = true
    delete_on_termination = true
  }
  
  monitoring = true
  
  metadata_options {
    http_tokens = "required"
    http_endpoint = "enabled"
  }
  
  tags = {
    Name = var.instance_name
  }
}

# Outputs
output "instance_id" {
  value = aws_instance.main.id
}

output "private_ip" {
  value = aws_instance.main.private_ip
}

output "security_group_id" {
  value = aws_security_group.instance.id
}
```

### 6. Lambda Validator
**Purpose:** Post-deployment validation

**Lambda Function:**
```python
import json
import boto3
import requests
from typing import Dict, List

ec2 = boto3.client('ec2')
cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    """
    Validates infrastructure deployment
    """
    print(f"Validating deployment: {json.dumps(event)}")
    
    resource_type = event['resource_type']
    resource_id = event['resource_id']
    validation_rules = event.get('validation_rules', [])
    
    errors = []
    
    if resource_type == 'ec2':
        errors.extend(validate_ec2_instance(resource_id, validation_rules))
    elif resource_type == 'rds':
        errors.extend(validate_rds_instance(resource_id, validation_rules))
    elif resource_type == 's3':
        errors.extend(validate_s3_bucket(resource_id, validation_rules))
    
    # Run common validations
    errors.extend(validate_tags(resource_type, resource_id))
    errors.extend(validate_monitoring(resource_type, resource_id))
    
    if errors:
        return {
            'validationStatus': 'FAILED',
            'validationErrors': errors
        }
    
    return {
        'validationStatus': 'PASSED',
        'message': f'All validations passed for {resource_type} {resource_id}'
    }


def validate_ec2_instance(instance_id: str, rules: List[Dict]) -> List[str]:
    """Validate EC2 instance configuration"""
    errors = []
    
    try:
        response = ec2.describe_instances(InstanceIds=[instance_id])
        instance = response['Reservations'][0]['Instances'][0]
        
        # Check instance state
        if instance['State']['Name'] != 'running':
            errors.append(f"Instance {instance_id} is not running")
        
        # Check if monitoring is enabled
        if not instance.get('Monitoring', {}).get('State') == 'enabled':
            errors.append(f"Detailed monitoring not enabled")
        
        # Check if encrypted
        for bdm in instance.get('BlockDeviceMappings', []):
            if not bdm['Ebs'].get('Encrypted', False):
                errors.append(f"EBS volume not encrypted")
        
        # Check IMDSv2
        metadata_options = instance.get('MetadataOptions', {})
        if metadata_options.get('HttpTokens') != 'required':
            errors.append(f"IMDSv2 not enforced")
        
        # Check tags
        tags = {tag['Key']: tag['Value'] for tag in instance.get('Tags', [])}
        required_tags = ['ManagedBy', 'Environment', 'JiraTicket']
        for tag in required_tags:
            if tag not in tags:
                errors.append(f"Missing required tag: {tag}")
        
        # Custom validation rules
        for rule in rules:
            if rule['type'] == 'instance_type':
                if instance['InstanceType'] not in rule['allowed_values']:
                    errors.append(f"Instance type {instance['InstanceType']} not allowed")
            
            elif rule['type'] == 'security_group':
                sg_ids = [sg['GroupId'] for sg in instance['SecurityGroups']]
                if not any(sg in sg_ids for sg in rule['required_groups']):
                    errors.append(f"Required security group not attached")
        
    except Exception as e:
        errors.append(f"Error validating EC2 instance: {str(e)}")
    
    return errors


def validate_tags(resource_type: str, resource_id: str) -> List[str]:
    """Validate resource tags"""
    errors = []
    
    required_tags = ['ManagedBy', 'Environment', 'JiraTicket', 'CreatedBy']
    
    try:
        if resource_type == 'ec2':
            response = ec2.describe_tags(
                Filters=[
                    {'Name': 'resource-id', 'Values': [resource_id]}
                ]
            )
            tags = {tag['Key']: tag['Value'] for tag in response['Tags']}
        
        for tag in required_tags:
            if tag not in tags:
                errors.append(f"Missing required tag: {tag}")
            elif not tags[tag]:
                errors.append(f"Tag {tag} has empty value")
    
    except Exception as e:
        errors.append(f"Error validating tags: {str(e)}")
    
    return errors


def validate_monitoring(resource_type: str, resource_id: str) -> List[str]:
    """Validate CloudWatch monitoring"""
    errors = []
    
    try:
        # Check if metrics exist
        response = cloudwatch.list_metrics(
            Dimensions=[
                {'Name': 'InstanceId', 'Value': resource_id}
            ]
        )
        
        if not response['Metrics']:
            errors.append(f"No CloudWatch metrics found for {resource_id}")
    
    except Exception as e:
        errors.append(f"Error validating monitoring: {str(e)}")
    
    return errors
```

## 📈 Metrics & Monitoring

**Key Metrics:**
- Provisioning time (end-to-end)
- Approval time (request to approval)
- Validation success rate
- Cost per provisioning
- Failed deployments

**Dashboards:**
- Real-time provisioning status
- Cost tracking by environment
- Resource utilization
- Approval workflow metrics

## 💰 Cost Optimization

**Automation Savings:**
- Manual provisioning: 2 hours × $50/hour = $100
- Automated: 5 minutes × $0 = $0
- **Savings per request: $100**

**Infrastructure Costs:**
- Azure DevOps: Free tier
- Lambda: $0.20 per 1M requests
- JIRA API: Included in license
- Teams: Included in M365

## 🚀 Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for step-by-step setup instructions.

## 📚 Related Documentation

- [Terraform Modules](../../code-samples/terraform-modules/)
- [Lambda Validator Code](../../code-samples/lambda-functions/validator.py)
- [Azure Pipeline Templates](../../code-samples/azure-pipelines/)

---

**Last Updated:** June 2026
**Version:** 1.5
**Status:** Production ✅

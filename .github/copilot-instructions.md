# AWS Certification Study Repository - AI Agent Guidelines

## Repository Overview
This repository contains study materials for AWS certifications, including practical code examples and detailed explanations. It's organized into two main sections:
- `code_v2025-10-27/`: Hands-on code samples (CloudFormation templates, shell scripts, HTML demos)
- `self-study/`: Comprehensive Markdown notes on AWS services

## Code Organization Patterns
- **Service-based structure**: Group files by AWS service (e.g., `ec2/`, `s3/`, `cloudformation/`)
- **Progressive examples**: Use numbered prefixes (0-, 1-, 2-) for sequential learning
- **Executable scripts**: Start with `#!/bin/bash`, include comments for educational context
- **CloudFormation**: Use YAML format with Parameters, Resources, Outputs sections

## Key Examples to Reference
- Simple EC2 instance: [code_v2025-10-27/cloudformation/0-just-ec2.yaml](code_v2025-10-27/cloudformation/0-just-ec2.yaml)
- EC2 with security groups: [code_v2025-10-27/cloudformation/1-ec2-with-sg-eip.yaml](code_v2025-10-27/cloudformation/1-ec2-with-sg-eip.yaml)
- S3 static website demo: [code_v2025-10-27/s3/index.html](code_v2025-10-27/s3/index.html)
- CLI script pattern: [code_v2025-10-27/cli/ec2-metadata.sh](code_v2025-10-27/cli/ec2-metadata.sh)

## Development Workflows
- **Script execution**: Make executable with `chmod +x script.sh`, run with `./script.sh`
- **CloudFormation validation**: `aws cloudformation validate-template --template-body file://template.yaml`
- **AWS CLI commands**: Assume configured credentials, use us-east-1 region by default
- **HTML demos**: Open in browser to test S3 static hosting or CORS functionality

## Content Conventions
- **Bilingual support**: Write the explanation in Korean, while keeping AWS-related terminology in English in md files. Include both Korean and English content where relevant
- **Markdown structure**: Use tables, code blocks, and hierarchical headings
- **Educational focus**: Include explanations, use cases, and exam-relevant details
- **Practical examples**: Ensure code is runnable and demonstrates real AWS scenarios
- **AWS SAA(Solution Architect Associate) quiz**: Create multiple-choice questions that are highly likely to appear on the SAA exam, based on past exam questions. Format should be like questions, answer, explantion. Questions should be in English, but Explanation should be in Korean.



## Adding New Content
1. Place code examples in `code_v2025-10-27/{service}/`
2. Add corresponding study notes in `self-study/{service}/`
3. Follow existing naming conventions and file structures
4. Include both implementation and explanation for certification prep

## Common Patterns
- EC2 metadata access via `http://169.254.169.254/latest/meta-data/`
- Security groups with specific port openings (22 for SSH, 80 for HTTP)
- Elastic IPs for static public addresses
- CORS configurations for cross-origin S3 requests
# Domain Monitor Deployment

Deployment configurations and CI/CD pipelines for the Domain Monitoring System.

## Project Overview

This repository contains Ansible playbooks, Jenkinsfile, and Kubernetes manifests for deploying the Domain Monitoring System. It automates the build, test, and deployment process across development and production environments.

## Repository Structure

```
domain-monitor-deploy/
├── ansible/
│   ├── ansible.cfg                # Ansible configuration
│   ├── inventory_aws_ec2.yaml     # Dynamic AWS inventory
│   ├── Jenkinsfile                # CI/CD pipeline definition
│   └── playbook.yaml              # Main Ansible playbook
├── .gitignore                     # Git ignore file
└── README.md                      # This documentation
```

## Components

### Ansible

The Ansible playbook automates the deployment of the Domain Monitoring System to AWS EC2 instances. It's triggered by the Jenkins pipeline and executed from the ansible agent EC2 instance.

Key components:
- `ansible.cfg`: Configuration for Ansible behavior
- `inventory_aws_ec2.yaml`: Dynamic inventory to discover AWS instances by tags
- `playbook.yaml`: Main deployment tasks

### Jenkins Pipeline

The Jenkinsfile defines a CI/CD pipeline with the following stages:
1. Clean Workspace
2. Clone Repository
3. Build Docker Images
4. Start Application Stack
5. Service Check
6. Selenium Tests
7. Push Images to Registry

## Setup and Usage

### Prerequisites

- Terraform infrastructure deployed (from domain-monitor-infra)
- AWS EC2 instances running
- Jenkins server configured with proper credentials

### Deployment Process

This repository is part of an automated CI/CD process:

1. Changes pushed to the main repositories trigger the Jenkins pipeline
2. Jenkins builds Docker images for frontend and backend
3. Images are tested in a staging environment
4. Ansible deploys the tested images to production servers

### Manual Deployment

If needed, you can manually trigger the Ansible deployment:

1. SSH into the Ansible agent EC2 instance
2. Clone this repository
3. Run the Ansible playbook:
   ```bash
   cd domain-monitor-deploy/ansible
   ansible-playbook -i inventory_aws_ec2.yaml playbook.yaml
   ```

## Ansible Playbook Details

The main playbook (`playbook.yaml`) performs the following tasks:

1. Sets up common configurations across all hosts
2. Configures server roles based on AWS EC2 tags:
   - Jenkins server configuration
   - Docker agents for building containers
   - Production servers for hosting the application

## Jenkins Pipeline Details

The Jenkins pipeline (`Jenkinsfile`) automates:

1. Building frontend and backend Docker images
2. Running tests to verify functionality
3. Deploying tested images to production servers

Key environment variables:
- `DOCKER_CREDENTIALS_ID`: For Docker Hub authentication
- `FE_IMAGE`: Frontend image repository
- `BE_IMAGE`: Backend image repository

## Integration with Infrastructure

This deployment repository works in conjunction with the infrastructure repository:

1. Terraform (in domain-monitor-infra) provisions the EC2 instances
2. EC2 instances are tagged with specific roles (jenkins, agent, production)
3. The Ansible inventory uses these tags to target specific instances
4. The deployment pipeline runs on the jenkins instance and uses the agent instance

### Ansible Agent Integration

This repository is designed to be cloned and executed by the Ansible agent role defined in the infrastructure repository:

1. The infrastructure repository contains an ansible_agent role at:
   `domain-monitor-infra/ansible/roles/ansible_agent/`

2. When the ansible_agent role runs, it:
   - Clones this repository to the agent EC2 instance
   - Sets up the necessary credentials and permissions
   - Makes the playbook available for execution from the Jenkins pipeline

3. The Jenkins pipeline then triggers the playbook execution on the agent via SSH or the Jenkins Ansible plugin

This connection ensures that the infrastructure setup and application deployment remain synchronized and follow the Infrastructure as Code principles.

## Troubleshooting

### Common Issues

#### Ansible Connection Problems
- Check that EC2 instances are running
- Verify security groups allow SSH access
- Ensure the inventory file correctly identifies hosts

#### Jenkins Pipeline Failures
- Check Jenkins console output for specific error messages
- Verify Docker credentials are correctly configured
- Ensure EC2 instances have sufficient resources

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -am 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Submit a pull request

## License

[MIT License](LICENSE)
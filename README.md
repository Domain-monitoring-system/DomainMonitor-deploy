# Domain Monitor Deployment

Deployment configurations and CI/CD pipelines for the Domain Monitoring System.

## Project Overview

This repository contains Ansible playbooks and Jenkinsfile for deploying the Domain Monitoring System. It automates the build, test, and deployment process across development and production environments.

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

## Test Integration

### Integration with Test Repository

The CI/CD pipeline integrates automated testing to ensure application quality before deployment. The pipeline uses tests from the domain-monitor-tests repository.

#### Test Execution in Pipeline

The Jenkinsfile includes a dedicated testing stage:

```groovy
stage('Selenium Test') {
    steps {
        script {
            sh """
                # Start Selenium container with host network
                docker run --network host \
                    -d \
                    --name selenium-test \
                    -e APP_URL=http://localhost:8080 \
                    -e WAIT_TIMEOUT=30 \
                    -e PYTHONUNBUFFERED=1 \
                    razielrey/selenium-tests:latest
                
                sleep 10
                
                # Run tests with output
                if ! docker exec selenium-test python3 -u run_tests.py; then
                    echo "Selenium tests failed - collecting debug info"
                    docker logs selenium-test
                    docker logs fe-app-${BUILD_TAG}
                    docker logs be-app-${BUILD_TAG}
                    exit 1
                fi
            """
        }
    }
}
```

#### Selenium Test Container

The pipeline uses a Docker container (`razielrey/selenium-tests`) that contains:
1. The test framework from the domain-monitor-tests repository
2. Chrome browser and WebDriver for UI testing
3. Python test dependencies

#### Test Flow

1. **Setup**: The pipeline starts frontend and backend containers to be tested
2. **Test Execution**: The Selenium container runs against these services
3. **Verification**: The `run_tests.py` script executes the core test suite
4. **Results**: A non-zero exit code fails the pipeline and prevents deployment

#### Test Categories Executed

The automated tests verify:
- User authentication (registration, login)
- Domain management functionality
- Scheduler operations
- Basic UI interactions

#### Test Artifacts and Reporting

Test logs and results are:
1. Captured in the Jenkins console output
2. Available in the test_logs directory within the container
3. Archived as build artifacts in case of failures

#### Handling Test Failures

If tests fail:
1. Debug information is collected from all containers
2. The pipeline stops, preventing deployment of potentially faulty code
3. Developers receive notifications about the failure

#### Test Environment Configuration

The test container can be configured with environment variables:
- `APP_URL`: URL of the application to test
- `WAIT_TIMEOUT`: Custom timeout for UI elements
- `PYTHONUNBUFFERED`: Ensures immediate log output

#### Adding Tests to the Pipeline

New tests are integrated by:
1. Adding test files to the domain-monitor-tests repository
2. Updating the `run_tests.py` script to include new test cases
3. Building a new version of the Selenium test container

### Performance Testing

For performance testing:
1. The Locust test file (`locustfile.py`) in the test repository can be used
2. Performance tests are typically run manually or as a separate job
3. Results are evaluated to identify performance regressions

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

## Integration with Infrastructure and Tests

This deployment repository works in conjunction with other repositories:

### Infrastructure Integration
1. Terraform (in domain-monitor-infra) provisions the EC2 instances
2. EC2 instances are tagged with specific roles (jenkins, agent, production)
3. The Ansible inventory uses these tags to target specific instances
4. The deployment pipeline runs on the jenkins instance and uses the agent instance

### Test Integration with domain-monitor-tests
1. Tests are maintained in the domain-monitor-tests repository
2. The Selenium test container is built from the test repository code
3. Tests are executed as part of the CI/CD pipeline before deployment
4. Test results determine whether deployment proceeds

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

#### Testing Issues
- Review test logs for specific failure details
- Check if the Selenium container can access the application
- Verify that test dependencies are correctly installed

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -am 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Submit a pull request

## License

[MIT License](LICENSE)
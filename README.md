# DevOps Project

A Jenkins pipeline that provisions an Azure VM using Terraform, configures a web server using Ansible, and deploys a static website.

## Setup
1. Install Docker, Terraform, Ansible, and Azure CLI.
2. Generate an SSH key pair.
3. Configure Azure credentials in Jenkins.
4. Run Jenkins in Docker.
5. Trigger the pipeline.

## Structure
- `terraform/`: Terraform scripts for VM provisioning.
- `ansible/`: Ansible playbooks for web server setup.
- `app/`: Static website files.
- `Jenkinsfile`: Pipeline definition.
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

## Docker Command
docker run -d `
  --name jenkins `
  -p 8080:8080 -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  -v //var/run/docker.sock:/var/run/docker.sock `
  -v (Get-Command terraform).Path:/usr/local/bin/terraform `
  -v (Get-Command ansible-playbook).Path:/usr/local/bin/ansible-playbook `
  -v C:/DevOpsProject:/workspace `
  -v C:/DevOpsProject/id_rsa:/root/.ssh/id_rsa `
  jenkins/jenkins:lts

## Troubleshooting SSH Authentication Issues with Ansible and Azure VM
#Problem Description
During the deployment of the Azure VM using Terraform and Ansible via Jenkins, we encountered persistent Permission denied (publickey) errors when Ansible attempted to connect to the VM for the Ansible Deploy stage. The issue manifested as Ansible failing to authenticate using the SSH private key, despite copying the key to the Jenkins container. Initial symptoms included:
- Ansible trying to use /root/.ssh/id_rsa instead of the correct path.
- Connection attempts to incorrect IP addresses (e.g., private IP 172.172.223.125 instead of the expected public IP like 4.246.220.197).

## Root Causes Investigated
- Key Path Mismatch: The Jenkinsfile initially pointed to /root/.ssh/id_rsa, while the key was copied to /var/jenkins_home/.ssh/id_rsa.
- Incorrect IP Address: Terraform’s public_ip output returned a private IP, causing Ansible to target the wrong address.
- Key Mismatch or Permissions: The private key’s permissions were 0777 on the host, preventing ssh-keygen from reading it, and potential mismatches with the VM’s
  authorized_keys.
- VM SSH Configuration: Possible misconfiguration in the VM’s SSH server settings.
- Network/Firewall Issues: Potential blocking of port 22 by Azure NSG or timing issues with VM readiness.

## Steps Taken to Resolve
1. Verified and Fixed Key Permissions:
   - Adjusted permissions on the host private key (/mnt/c/Users/hp/.ssh/id_rsa) from 0777 to 600 using chmod in WSL or Windows Security settings to allow ssh
     keygen -y to extract the public key for comparison.
   - Copied the key to the Jenkins container at /var/jenkins_home/.ssh/id_rsa and set chmod 600 and chown jenkins:jenkins.
2. Updated Jenkinsfile:
   - Modified the Ansible Inventory stage to use ansible_ssh_private_key_file: /var/jenkins_home/.ssh/id_rsa instead of /root/.ssh/id_rsa.
   - Added a Debug IP stage to log the IP from public_ip.txt for troubleshooting.
3. Debugged IP Issue:
   - Confirmed public_ip.txt contained a private IP (172.172.223.125) instead of the expected public IP.
   - Considered adding an Azure CLI step to fetch the correct public IP but focused on fixing Terraform output first.
4. Attempted Password Authentication:
   - Enabled PasswordAuthentication yes in the VM’s /etc/ssh/sshd_config via Azure Serial Console.
   - Set a password for azureuser using sudo passwd azureuser.
   - Updated the Jenkinsfile to use ansible_user, ansible_password (stored as a Jenkins credential), and ansible_connection: ansible.builtin.paramiko for password
     based SSH.
5. Proposed New Key Pair Generation:
   - Planned to generate a new RSA key pair (ssh-keygen -t rsa -b 4096 -C "hp@ROHALaptop" -f /mnt/c/DevOpsProject/id_rsa).
   - Intended to update variables.tf with the new public key and copy the private key to the container, re-running Terraform and the pipeline.
6. Additional Diagnostics:
   - Suggested manual SSH tests from the container (ssh -i /var/jenkins_home/.ssh/id_rsa -o StrictHostKeyChecking=no azureuser@<IP>).
   - Recommended checking VM SSH logs (/var/log/auth.log) and NSG rules if issues persisted.

## Current Status
The issue remains unresolved, with the latest attempt showing Permission denied (publickey) on public IP 4.246.220.197. The next step involves generating a new key pair and verifying VM SSH configuration, or reverting to password authentication if key-based auth continues to fail.

## Pipeline
![image](https://github.com/user-attachments/assets/78019981-8b8c-4694-abda-be0888ff2469)

![image](https://github.com/user-attachments/assets/51003416-d466-4d0f-8bcf-07b755e54e4f)

![image](https://github.com/user-attachments/assets/72cb7775-f500-4c78-9377-26937fbd6d35)
![image](https://github.com/user-attachments/assets/33a84784-62c3-4a46-bc63-ab0c5e3f8eb1)
![image](https://github.com/user-attachments/assets/d4d3d8c9-00a8-49fc-8220-f0b60ec08b34)
![image](https://github.com/user-attachments/assets/7f583d4a-6669-4acc-a064-bf858098bdcf)
![image](https://github.com/user-attachments/assets/5c0ac21b-aed0-4a3c-a26b-7bef1358edc8)

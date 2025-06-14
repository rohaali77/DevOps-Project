pipeline {
    agent any
    environment {
        ARM_CLIENT_ID = credentials('arm-client-id')
        ARM_CLIENT_SECRET = credentials('arm-client-secret')
        ARM_TENANT_ID = credentials('arm-tenant-id')
        ARM_SUBSCRIPTION_ID = credentials('arm-subscription-id')
    }
    stages {
        stage('Debug Credentials') {
            steps {
                sh 'echo $ARM_CLIENT_ID'
                sh 'echo $ARM_SUBSCRIPTION_ID'
                sh 'echo $ARM_TENANT_ID'
                sh 'echo $ARM_CLIENT_SECRET'
            }
        }
        stage('Checkout') {
            steps {
                git url: 'https://github.com/rohaali77/DevOps-Project', branch: 'main'
            }
        }
        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh '''
                        terraform apply -auto-approve
                        terraform output -raw public_ip > ../ansible/public_ip.txt
                    '''
                }
            }
        }
        stage('Wait for VM') {
            steps {
                sh '''
                    PUBLIC_IP=$(cat ansible/public_ip.txt)
                    echo "Waiting for VM at $PUBLIC_IP to be ready..."
                    until nc -zv $PUBLIC_IP 22; do
                        echo "SSH not ready, waiting..."
                        sleep 10
                    done
                    echo "VM is up, proceeding with Ansible..."
                '''
            }
        }
        stage('Ansible Inventory') {
            steps {
                dir('ansible') {
                    sh '''
                        echo "---" > inventory.yml
                        echo "all:" >> inventory.yml
                        echo "  hosts:" >> inventory.yml
                        echo "    web:" >> inventory.yml
                        echo "      ansible_host: $(cat public_ip.txt)" >> inventory.yml
                        echo "      ansible_user: azureuser" >> inventory.yml
                        echo "      ansible_ssh_private_key_file: ~/.ssh/id_rsa" >> inventory.yml
                    '''
                }
            }
        }
        stage('Ansible Deploy') {
            steps {
                dir('ansible') {
                    sh '/usr/local/bin/ansible-playbook -i inventory.yml install_web.yml --extra-vars "ansible_ssh_common_args=\'-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null\'"'
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                sh '''
                    PUBLIC_IP=$(cat ansible/public_ip.txt)
                    curl http://$PUBLIC_IP || echo "Verification failed, check manually"
                '''
            }
        }
    }
    post {
        always {
            dir('terraform') {
                sh 'terraform destroy -auto-approve'
            }
        }
    }
}

pipeline {
    agent any
    environment {
        ARM_CLIENT_ID = credentials('ARM_CLIENT_ID')
        ARM_SUBSCRIPTION_ID = credentials('ARM_SUBSCRIPTION_ID')
        ARM_TENANT_ID = credentials('ARM_TENANT_ID')
        ARM_CLIENT_SECRET = credentials('ARM_CLIENT_SECRET')
    }
    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }
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
                    sh 'terraform apply -auto-approve'
                    sh 'terraform output -raw public_ip > ../ansible/public_ip.txt'
                }
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
                        cat public_ip.txt >> inventory.yml
                        echo "      ansible_host: \$(cat public_ip.txt)" >> inventory.yml
                        echo "      ansible_user: azureuser" >> inventory.yml
                        echo "      ansible_ssh_private_key_file: ~/.ssh/id_rsa" >> inventory.yml
                    '''
                }
            }
        }
        stage('Ansible Deploy') {
            steps {
                dir('ansible') {
                    sh '/usr/local/bin/ansible-playbook -i inventory.yml install_web.yml'
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                dir('ansible') {
                    sh 'curl http://$(cat public_ip.txt) || echo "Verification failed, check manually"'
                }
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

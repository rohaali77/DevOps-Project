pipeline {
  agent any
  environment {
    AZURE_CREDENTIALS = credentials('azure-credentials')
    SSH_CREDENTIALS = credentials('ssh-key')
  }
  stages {
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
          sh 'ansible-playbook -i inventory.yml install_web.yml'
        }
      }
    }
    stage('Verify Deployment') {
      steps {
        sh '''
          PUBLIC_IP=$(cat ansible/public_ip.txt)
          curl http://$PUBLIC_IP
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

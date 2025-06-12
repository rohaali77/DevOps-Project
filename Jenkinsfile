pipeline {
  agent any

  environment {
    SSH_KEY_PATH = "${WORKSPACE}/id_rsa"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git url: 'https://github.com/rohaali77/DevOps-Project.git', branch: 'main'
      }
    }

    stage('Provision Infrastructure') {
      steps {
        withAzureCredentials(
          credentialsId: 'azure-credentials',
          subscriptionIdVariable: 'ARM_SUBSCRIPTION_ID',
          clientIdVariable: 'ARM_CLIENT_ID',
          clientSecretVariable: 'ARM_CLIENT_SECRET',
          tenantIdVariable: 'ARM_TENANT_ID'
        ) {
          dir('terraform') {
            sh '''
              export ARM_CLIENT_ID=$ARM_CLIENT_ID
              export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
              export ARM_TENANT_ID=$ARM_TENANT_ID
              export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

              terraform init
              terraform apply -auto-approve
              terraform output -raw public_ip > ../ansible/public_ip.txt
            '''
          }
        }
      }
    }

    stage('Prepare Ansible Inventory') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY')]) {
          dir('ansible') {
            sh '''
              cp $SSH_KEY ../id_rsa
              chmod 600 ../id_rsa

              PUBLIC_IP=$(cat public_ip.txt)

              cat <<EOF > inventory.yml
all:
  hosts:
    web:
      ansible_host: $PUBLIC_IP
      ansible_user: azureuser
      ansible_ssh_private_key_file: ../id_rsa
EOF
            '''
          }
        }
      }
    }

    stage('Deploy Web App with Ansible') {
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
          curl -f http://$PUBLIC_IP
        '''
      }
    }
  }

  post {
    always {
      withAzureCredentials(
        credentialsId: 'azure-credentials',
        subscriptionIdVariable: 'ARM_SUBSCRIPTION_ID',
        clientIdVariable: 'ARM_CLIENT_ID',
        clientSecretVariable: 'ARM_CLIENT_SECRET',
        tenantIdVariable: 'ARM_TENANT_ID'
      ) {
        dir('terraform') {
          sh '''
            export ARM_CLIENT_ID=$ARM_CLIENT_ID
            export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
            export ARM_TENANT_ID=$ARM_TENANT_ID
            export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

            terraform destroy -auto-approve
          '''
        }
      }
    }
  }
}

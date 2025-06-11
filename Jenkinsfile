pipeline {
  agent any

  environment {
    SSH_KEY_PATH = "${WORKSPACE}/id_rsa"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/rohaali77/DevOps-Project', branch: 'main'
      }
    }

    stage('Setup Credentials') {
      steps {
        withAzureCredentials(
          credentialsId: 'azure-credentials',
          subscriptionIdVariable: 'ARM_SUBSCRIPTION_ID',
          clientIdVariable: 'ARM_CLIENT_ID',
          clientSecretVariable: 'ARM_CLIENT_SECRET',
          tenantIdVariable: 'ARM_TENANT_ID'
        ) {
          withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY')]) {
            sh '''
              echo "Azure credentials loaded..."
              echo "ARM_CLIENT_ID: $ARM_CLIENT_ID"
              echo "ARM_TENANT_ID: $ARM_TENANT_ID"
              echo "ARM_SUBSCRIPTION_ID: $ARM_SUBSCRIPTION_ID"

              # Copy the SSH key to the workspace
              cp $SSH_KEY $SSH_KEY_PATH
              chmod 600 $SSH_KEY_PATH

              export ARM_CLIENT_ID=$ARM_CLIENT_ID
              export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
              export ARM_TENANT_ID=$ARM_TENANT_ID
              export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

              cd terraform
              terraform init
              terraform apply -auto-approve
              terraform output -raw public_ip > ../ansible/public_ip.txt
            '''
          }
        }
      }
    }

    stage('Ansible Inventory') {
      steps {
        dir('ansible') {
          sh '''
            PUBLIC_IP=$(cat public_ip.txt)
            cat <<EOF > inventory.yml
all:
  hosts:
    web:
      ansible_host: $PUBLIC_IP
      ansible_user: azureuser
      ansible_ssh_private_key_file: ${SSH_KEY_PATH}
EOF
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
        sh '''
          export ARM_CLIENT_ID=$ARM_CLIENT_ID
          export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
          export ARM_TENANT_ID=$ARM_TENANT_ID
          export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

          cd terraform
          terraform destroy -auto-approve
        '''
      }
    }
  }
}

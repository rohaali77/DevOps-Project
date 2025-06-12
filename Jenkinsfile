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

    stage('Terraform Provision') {
      steps {
        withCredentials([
          string(credentialsId: 'azure-client-id', variable: 'ARM_CLIENT_ID'),
          string(credentialsId: 'azure-client-secret', variable: 'ARM_CLIENT_SECRET'),
          string(credentialsId: 'azure-tenant-id', variable: 'ARM_TENANT_ID'),
          string(credentialsId: 'azure-subscription-id', variable: 'ARM_SUBSCRIPTION_ID'),
          sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY_FILE')
        ]) {
          sh '''
            # Set up SSH key
            cp $SSH_KEY_FILE $SSH_KEY_PATH
            chmod 600 $SSH_KEY_PATH

            # Export Azure environment variables for Terraform
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
      ansible_ssh_private_key_file: ../id_rsa
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
          curl http://$PUBLIC_IP
        '''
      }
    }
  }

  post {
    always {
      withCredentials([
        string(credentialsId: 'azure-client-id', variable: 'ARM_CLIENT_ID'),
        string(credentialsId: 'azure-client-secret', variable: 'ARM_CLIENT_SECRET'),
        string(credentialsId: 'azure-tenant-id', variable: 'ARM_TENANT_ID'),
        string(credentialsId: 'azure-subscription-id', variable: 'ARM_SUBSCRIPTION_ID')
      ]) {
        dir('terraform') {
          sh '''
            export ARM_CLIENT_ID=$ARM_CLIENT_ID
            export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
            export ARM_TENANT_ID=$ARM_TENANT_ID
            export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

            terraform destroy -auto-approve || true
          '''
        }
      }
    }
  }
}

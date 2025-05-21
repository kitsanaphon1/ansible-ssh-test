pipeline {
  agent any

  environment {
    ANSIBLE_HOST = "4.145.84.26"    // แก้ตาม IP Ansible VM
    SSH_USER = "boho"
  }

  stages {
    stage('🚀 Provision Azure VM via Ansible') {
      steps {
        withCredentials([
          string(credentialsId: 'AZURE_CLIENT_ID',         variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'AZURE_SECRET',            variable: 'AZURE_SECRET'),
          string(credentialsId: 'AZURE_TENANT',            variable: 'AZURE_TENANT'),
          string(credentialsId: 'AZURE_SUBSCRIPTION_ID',   variable: 'AZURE_SUBSCRIPTION_ID'),
          sshUserPrivateKey(credentialsId: 'ssh-ansible-agent', keyFileVariable: 'PRIVATE_KEY')
        ]) {
          script {
            def publicKey = sh(script: "ssh-keygen -y -f $PRIVATE_KEY", returnStdout: true).trim()

            sh """
              ssh -o StrictHostKeyChecking=no ${SSH_USER}@${ANSIBLE_HOST} <<'EOF'
                set -e
                echo "🟢 Activating Ansible venv..."
                source ~/ansible-env/bin/activate

                echo "🔐 Exporting Azure credentials..."
                export AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
                export AZURE_SECRET=${AZURE_SECRET}
                export AZURE_TENANT=${AZURE_TENANT}
                export AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
                export PUBLIC_KEY="${publicKey}"

                echo "📁 Switching to playbook directory..."
                cd ~/ANSIBLE-SSH-TEST/playbooks

                echo "🚀 Running Ansible playbook..."
                ansible-playbook create-linux-vm.yaml
              EOF
            """
          }
        }
      }
    }
  }
}

pipeline {
  agent any

  environment {
    ANSIBLE_HOST = "4.145.84.26"  // ✅ IP ของเครื่อง Ansible
    SSH_USER     = "boho"
    GIT_REPO     = "https://github.com/kitsanaphon1/ansible-ssh-test.git" // ✅ แก้เป็น repo จริง
    PROJECT_DIR  = "ansible-ssh-test"
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
          sshagent(['ssh-ansible-agent']) {
            sh '''
              echo "🔐 สร้าง Public Key จาก PRIVATE_KEY"
              PUBLIC_KEY=$(ssh-keygen -y -f "$PRIVATE_KEY")

              echo "🔗 SSH ไปยังเครื่อง Ansible..."
              ssh -o StrictHostKeyChecking=no ${SSH_USER}@${ANSIBLE_HOST} <<'EOF'
                set -e
                echo "✅ Activate venv"
                source ~/ansible-env/bin/activate

                echo "📦 Export Azure Credentials"
                export AZURE_CLIENT_ID='${AZURE_CLIENT_ID}'
                export AZURE_SECRET='${AZURE_SECRET}'
                export AZURE_TENANT='${AZURE_TENANT}'
                export AZURE_SUBSCRIPTION_ID='${AZURE_SUBSCRIPTION_ID}'
                export PUBLIC_KEY="${PUBLIC_KEY}"

                echo "📂 Clone git project ถ้ายังไม่มี"
                cd ~
                if [ ! -d "${PROJECT_DIR}" ]; then
                  git clone ${GIT_REPO}
                else
                  cd ${PROJECT_DIR}
                  git pull
                  cd ..
                fi

                echo "📂 ไปยัง playbook directory"
                cd ${PROJECT_DIR}/playbooks

                echo "🚀 Run playbook"
                ansible-playbook create-linux-vm.yaml
              EOF
            '''
          }
        }
      }
    }
  }
}

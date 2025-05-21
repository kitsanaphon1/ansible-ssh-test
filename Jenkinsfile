pipeline {
  agent any

  environment {
    GIT_BRANCH    = "dev"
    ANSIBLE_HOST  = "4.145.84.26"
    SSH_USER      = "boho"
    GIT_REPO      = "https://github.com/kitsanaphon1/ansible-ssh-test.git"
    PROJECT_DIR   = "ansible-ssh-test"
  }

  stages {
    stage('🚀 Provision Azure VM via Ansible') {
      steps {
        withCredentials([
          string(credentialsId: 'AZURE_CLIENT_ID',       variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'AZURE_SECRET',          variable: 'AZURE_SECRET'),
          string(credentialsId: 'AZURE_TENANT',          variable: 'AZURE_TENANT'),
          string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'AZURE_SUBSCRIPTION_ID'),
          sshUserPrivateKey(credentialsId: 'ssh-ansible-agent', keyFileVariable: 'PRIVATE_KEY')
        ]) {
          sshagent(['ssh-ansible-agent']) {
            sh """
              echo "🔐 สร้าง Public Key จาก PRIVATE_KEY"
              PUBLIC_KEY=\$(ssh-keygen -y -f "$PRIVATE_KEY")

              echo "📄 สร้างสคริปต์ชั่วคราวสำหรับ remote"
              cat > run_ansible_remote.sh <<EOF
#!/bin/bash
set -e

echo "✅ Activate Python venv"
source /home/boho/ansible-env/bin/activate

echo "📦 Export Azure Credentials"
export AZURE_CLIENT_ID='${AZURE_CLIENT_ID}'
export AZURE_SECRET='${AZURE_SECRET}'
export AZURE_TENANT='${AZURE_TENANT}'
export AZURE_SUBSCRIPTION_ID='${AZURE_SUBSCRIPTION_ID}'
export PUBLIC_KEY="\${PUBLIC_KEY}"

echo "📂 เตรียม Git Project"
cd ~
if [ ! -d "${PROJECT_DIR}" ]; then
  git clone "${GIT_REPO}"
fi

cd "${PROJECT_DIR}"
git fetch origin
git checkout -B ${GIT_BRANCH} origin/${GIT_BRANCH}
git pull origin ${GIT_BRANCH}

echo "🚀 Run playbook พร้อม config-dev.yaml"
cd playbooks
ansible-playbook create-linux-vm.yaml -e "@../config/config-dev.yaml"
EOF

              echo "📡 ส่ง script ไปยังเครื่อง Ansible"
              scp -o StrictHostKeyChecking=no run_ansible_remote.sh ${SSH_USER}@${ANSIBLE_HOST}:/tmp/

              echo "🚀 รัน script บน Ansible VM"
              ssh -o StrictHostKeyChecking=no ${SSH_USER}@${ANSIBLE_HOST} 'bash /tmp/run_ansible_remote.sh'
            """
          }
        }
      }
    }
  }
}

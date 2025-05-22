pipeline {
  agent any

  environment {
    GIT_BRANCH    = "dev"                     // Branch ที่จะ checkout
    ANSIBLE_HOST  = "4.145.84.26"             // IP ของเครื่อง Ansible VM
    SSH_USER      = "boho"                    // SSH user ของ Ansible VM
    GIT_REPO      = "https://github.com/kitsanaphon1/ansible-ssh-test.git"
    PROJECT_DIR   = "ansible-ssh-test"
    DESTROY_MODE  = "false"                   // "true" = ลบ VM, "false" = สร้าง VM
  }

  stages {
    stage('🚀 Provision หรือ Destroy VM') {
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
              echo "🔐 สร้าง Public Key จาก Jenkins Credential"
              PUBLIC_KEY=\$(ssh-keygen -y -f "$PRIVATE_KEY")

              echo "📄 เตรียมสคริปต์ Remote"
              cat > run_ansible_remote.sh <<EOF
#!/bin/bash
set -e
source /home/boho/ansible-env/bin/activate

export AZURE_CLIENT_ID="${AZURE_CLIENT_ID}"
export AZURE_SECRET="${AZURE_SECRET}"
export AZURE_TENANT="${AZURE_TENANT}"
export AZURE_SUBSCRIPTION_ID="${AZURE_SUBSCRIPTION_ID}"
export PUBLIC_KEY="\${PUBLIC_KEY}"

cd ~

if [ ! -d "${PROJECT_DIR}" ]; then
  git clone "${GIT_REPO}"
fi

cd "${PROJECT_DIR}"
git fetch origin
git checkout -B ${GIT_BRANCH} origin/${GIT_BRANCH}
git pull origin ${GIT_BRANCH}
cd playbooks

if [ "${DESTROY_MODE}" = "true" ]; then
  echo "🔥 ลบ VM ทั้งหมด"
  ansible-playbook destroy-linux-vm.yaml -e "@../config/config-dev.yaml"
else
  echo "🚀 สร้าง VM"
  ansible-playbook create-linux-vm.yaml \\
    -e "@../config/config-dev.yaml" \\
    -e "admin_ssh_public_key=\$PUBLIC_KEY"
fi
EOF

              echo "📡 ส่ง script ไปยัง Ansible VM"
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

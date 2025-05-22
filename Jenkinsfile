pipeline {
  agent any  // 🧑‍💻 ใช้ Jenkins agent อะไรก็ได้ที่มีอยู่

  environment {
    // 🔧 ตั้งค่าพื้นฐานที่จำเป็น
    GIT_BRANCH    = "dev"   // Branch ที่จะ checkout จาก Git
    ANSIBLE_HOST  = "4.145.84.26"  // IP ของเครื่องที่รัน Ansible
    SSH_USER      = "boho"         // User สำหรับ SSH เข้า Ansible VM
    GIT_REPO      = "https://github.com/kitsanaphon1/ansible-ssh-test.git"  // Git repo playbook
    PROJECT_DIR   = "ansible-ssh-test"  // โฟลเดอร์ project
    DESTROY_MODE  = "true"  // ✅ "true" = ลบ VM, "false" = สร้าง VM
  }

  stages {
    stage('🚀 Provision หรือ Destroy VM') {
      steps {
        // 🔐 ดึง Credential ทั้ง Azure และ SSH
        withCredentials([
          string(credentialsId: 'AZURE_CLIENT_ID',       variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'AZURE_SECRET',          variable: 'AZURE_SECRET'),
          string(credentialsId: 'AZURE_TENANT',          variable: 'AZURE_TENANT'),
          string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'AZURE_SUBSCRIPTION_ID'),
          sshUserPrivateKey(credentialsId: 'ssh-ansible-agent', keyFileVariable: 'PRIVATE_KEY')
        ]) {
          // ✅ ใช้ SSH Agent เพื่อ auth กับ Ansible VM
          sshagent(['ssh-ansible-agent']) {
            sh """
              echo "🔐 สร้าง Public Key จาก PRIVATE_KEY"
              PUBLIC_KEY=\$(ssh-keygen -y -f "$PRIVATE_KEY")

              echo "📄 เตรียมสคริปต์ Remote"
              cat > run_ansible_remote.sh <<EOF
#!/bin/bash
set -e
source /home/boho/ansible-env/bin/activate

# 🧠 Export ตัวแปร Azure credential ให้ Ansible ใช้ได้
export AZURE_CLIENT_ID="${AZURE_CLIENT_ID}"
export AZURE_SECRET="${AZURE_SECRET}"
export AZURE_TENANT="${AZURE_TENANT}"
export AZURE_SUBSCRIPTION_ID="${AZURE_SUBSCRIPTION_ID}"
export PUBLIC_KEY="\${PUBLIC_KEY}"

cd ~

# 📥 Clone โปรเจกต์จาก Git ถ้ายังไม่มี
if [ ! -d "${PROJECT_DIR}" ]; then
  git clone "${GIT_REPO}"
fi

cd "${PROJECT_DIR}"
git fetch origin
git checkout -B ${GIT_BRANCH} origin/${GIT_BRANCH}
git pull origin ${GIT_BRANCH}
cd playbooks

# 🔁 เช็ค DESTROY_MODE ว่าจะ 'ลบ' หรือ 'สร้าง' VM
if [ "${DESTROY_MODE}" = "true" ]; then
  echo "🔥 ลบ VM ทั้งหมด"
  ansible-playbook destroy-linux-vm.yaml -e "@../config/config-dev.yaml"
else
  echo "🚀 สร้าง VM"
  ansible-playbook create-linux-vm.yaml -e "@../config/config-dev.yaml"
fi
EOF

              # 📡 ส่งสคริปต์ไปยังเครื่อง Ansible
              scp -o StrictHostKeyChecking=no run_ansible_remote.sh ${SSH_USER}@${ANSIBLE_HOST}:/tmp/

              # 🚀 รันสคริปต์บนเครื่อง Ansible
              ssh -o StrictHostKeyChecking=no ${SSH_USER}@${ANSIBLE_HOST} 'bash /tmp/run_ansible_remote.sh'
            """
          }
        }
      }
    }
  }
}

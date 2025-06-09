pipeline {
  agent any

  environment {
    ANSIBLE_HOST_KEY_CHECKING = 'False'
    ANSIBLE_FORCE_COLOR = 'true'
  }

  stages {
    stage('Checkout Code') {
      steps {
        deleteDir()
        checkout scm
      }
    }

    stage('Deploy with Ansible') {
      steps {
        withCredentials([file(credentialsId: 'aws_ec2_key', variable: 'PEM_KEY')]) {
          sh '''
            echo "Using SSH Key at: $PEM_KEY"
            chmod 600 $PEM_KEY
            
            # Create minimal inventory file
            cat > inventory.yml <<EOF
all:
  children:
    target:
      hosts:
        rails-server:
          ansible_host: 44.201.206.78
          ansible_user: ubuntu
          ansible_ssh_private_key_file: $PEM_KEY
          ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

        rails-server-2:
          ansible_host: 44.203.123.99
          ansible_user: rpx
          ansible_ssh_private_key_file: $jenkins_key
          ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
EOF

            # Run the playbook directly on target server
            ansible-playbook -i inventory.yml playbook.yml -v
          '''
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}

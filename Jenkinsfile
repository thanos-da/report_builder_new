pipeline {
  agent any

  environment {
    ANSIBLE_HOST_KEY_CHECKING = 'False'
  }

  stages {
    stage('Checkout Code') {
      steps {
        git url: 'https://github.com/thanos-da/report_builder_new.git', branch: 'main'
      }
    }

    stage('Deploy with Ansible') {
      steps {
        withCredentials([file(credentialsId: 'aws_ec2_key', variable: 'PEM_KEY')]) {
          withEnv(["SSH_USER=ubuntu"]) {
            sh '''
              echo "Using SSH Key at: $PEM_KEY"
              # Generate dynamic inventory file
              cat > inventory.yml <<EOF
all:
  children:
    rails_servers:
      hosts:
        rails-server:
          ansible_host: 54.173.135.9
          ansible_user: ubuntu
          ansible_ssh_private_key_file: $PEM_KEY
EOF

              # Run the Ansible playbook
              ansible-playbook -i inventory.yml playbook.yml
            '''
          }
        }
      }
    }
  }
}

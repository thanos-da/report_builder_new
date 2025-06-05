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
            echo "Using SSH Key at: $SSH_KEY"
            
            # Optionally create or update inventory dynamically
            cat <<EOF > inventory.yml
all:
  hosts:
    rails-server:
      ansible_host: 54.91.234.22
      ansible_user: $SSH_USER
      ansible_ssh_private_key_file: $SSH_KEY
  children:
    rails_servers:
      hosts:
        rails-server:
EOF

            # Run Ansible playbook
            ansible-playbook -i inventory.yml playbook.yml
          '''
          }
        }
      }
    }
  }
}

pipeline {
  agent any

  environment {
    ANSIBLE_HOST_KEY_CHECKING = 'False'
    ANSIBLE_FORCE_COLOR = 'true'  // Makes Ansible output more readable
  }

  stages {
    stage('Checkout Code') {
      steps {
        script {
          checkout([
            $class: 'GitSCM',
            branches: [[name: 'main']],
            extensions: [],
            userRemoteConfigs: [[
              url: 'https://github.com/thanos-da/report_builder_new.git',
            ]]
          ])
        }
      }
    }

    stage('Verify Files') {
      steps {
        sh '''
          echo "Current workspace contents:"
          ls -la
          echo "Checking playbook exists:"
          test -f playbook.yml && echo "Playbook found" || echo "Playbook missing"
        '''
      }
    }

    stage('Deploy with Ansible') {
      steps {
        withCredentials([file(credentialsId: 'aws_ec2_key', variable: 'PEM_KEY')]) {
          script {
            // Create the inventory file
            writeFile file: 'inventory.yml', text: """
all:
  children:
    rails_servers:
      hosts:
        rails-server:
          ansible_host: 44.201.206.78
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ${PEM_KEY}
          ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
"""
            // Verify the inventory file
            sh 'cat inventory.yml'
            
            // Run the playbook with verbose output
            sh '''
              echo "Using SSH Key at: $PEM_KEY"
              ansible-playbook -i inventory.yml playbook.yml -v
            '''
          }
        }
      }
    }
  }

  post {
    always {
      cleanWs() 
    }
    failure {
      emailext (
        subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: "Check console output at ${env.BUILD_URL}",
        to: 'your-team@example.com'
      )
    }
  }
}

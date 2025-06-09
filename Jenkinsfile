pipeline {
  agent any

  environment {
    ANSIBLE_HOST_KEY_CHECKING = 'False'
    ANSIBLE_FORCE_COLOR = 'true'
  }

  stages {
    stage('Checkout Code') {
      steps {
        deleteDir()  // Clean workspace before checkout
        checkout scm  // Uses the SCM configuration from the job
      }
    }

    stage('Verify Files') {
      steps {
        sh '''
          echo "Workspace contents:"
          ls -la
          test -f playbook.yml || exit 1
        '''
      }
    }

    stage('Deploy with Ansible') {
      steps {
        withCredentials([file(credentialsId: 'aws_ec2_key', variable: 'PEM_KEY')]) {
          sh '''
            echo "Using SSH Key at: $PEM_KEY"
            chmod 600 $PEM_KEY
            
            # Create an inventory file without Groovy interpolation
            cat > inventory.yml <<'EOF'
all:
  hosts:
    rails-server:
      ansible_host: 44.201.206.78
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ${PEM_KEY}
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
EOF

            # Test SSH connection first
            ssh -i $PEM_KEY -o StrictHostKeyChecking=no ubuntu@44.201.206.78 exit
            
            # Run playbook
            ansible-playbook -i inventory.yml playbook.yml -v
          '''
        }
      }
    }
  }

  post {
    always {
      cleanWs()
      script {
        // Only send email if failed and email is configured
        if(currentBuild.result == 'FAILURE' && env.EMAIL_RECIPIENTS) {
          emailext(
            subject: "FAILED: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
            body: "Check console output at ${env.BUILD_URL}",
            to: env.EMAIL_RECIPIENTS
          )
        }
      }
    }
  }
}

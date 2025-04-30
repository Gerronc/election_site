pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"  // Or ubuntu, depending on your AMI
        EC2_HOST = "34.201.25.109"
        EC2_KEY = credentials('ec2-ssh-private-key')  // Jenkins credential with SSH private key
        PROJECT_DIR = "/home/ubuntu/pytests"  // Path to your Django app
        
    }

    //triggers {
      //  githubPush()  // Enables webhook triggering
    //}
    
    stages {
        stage('Update Code on EC2') {
            steps {
                script {
                    // Use SSH to run commands on the EC2 instance
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            cd ${PROJECT_DIR}
                            git clone https://github.com/kestonbhola/django_polls.git
                            python3 -m venv comp314
                            source comp314/bin/activate
                            pip install -r requirements.txt
                            python3 manage.py migrate
                            python3 manage.py collectstatic --noinput
                            sudo systemctl restart gunicorn
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Code updated and app restarted successfully on EC2!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}

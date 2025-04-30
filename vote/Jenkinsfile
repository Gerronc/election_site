pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"  // Assuming you're using the default Ubuntu AMI
        EC2_HOST = "3.15.145.112"
        EC2_KEY = credentials('ec2-ssh-private-key')  // Replace with your actual Jenkins credential ID
        PROJECT_DIR = "/home/ubuntu/your_project_directory"  // Change to your actual Django project folder
    }

    triggers {
        githubPush()  // This will trigger the job when you push to GitHub (requires webhook setup)
    }

    stages {
        stage('Deploy Django App') {
            steps {
                script {
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            cd ${PROJECT_DIR}
                            git pull origin main
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
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}

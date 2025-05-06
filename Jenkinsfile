pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.137.162.207"
        EC2_KEY = credentials('ec2-ssh-private-key')
        PROJECT_DIR = "/home/ubuntu/election_site"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('🔧 SSH Test to EC2') {
            steps {
                script {
                    echo "Attempting SSH connection to EC2..."
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'echo "✅ EC2 SSH connection successful!"'
                        """
                    }
                }
            }
        }

        stage('🚀 Deploy Django App') {
            steps {
                script {
                    echo "🟡 Starting deploy stage..."
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                            set -x
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                echo "✅ Connected to EC2"

                                cd ${PROJECT_DIR} || { echo "❌ Failed to cd into project directory"; exit 1; }

                                git pull origin main
                                source venv/bin/activate
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
            echo "❌ Deployment failed. Check the console output for details."
        }
    }
}

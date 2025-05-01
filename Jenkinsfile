pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"  // Default Ubuntu AMI user
        EC2_HOST = "3.15.145.112"
        EC2_KEY = credentials('ec2-ssh-private-key')  // Your Jenkins credential ID
        PROJECT_DIR = "/home/ubuntu/election_site"  // Update as needed
    }

    triggers {
        githubPush()  // Requires GitHub webhook to be configured
    }

    stages {
        stage('Deploy Django App') {
            steps {
                script {
                    echo "🟡 Starting deploy stage..."

                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        def result = sh(script: """
                            set -x  # Enable command trace
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                echo "✅ Connected to EC2"

                                cd ${PROJECT_DIR} || { echo "❌ Failed to cd into project directory"; exit 1; }

                                echo "🔄 Pulling latest code from GitHub..."
                                git pull origin main

                                echo "🐍 Activating virtual environment..."
                                source venv/bin/activate

                                echo "📦 Installing dependencies..."
                                pip install -r requirements.txt

                                echo "🛠 Applying database migrations..."
                                python3 manage.py migrate

                                echo "📁 Collecting static files..."
                                python3 manage.py collectstatic --noinput

                                echo "🚀 Restarting Gunicorn..."
                                sudo systemctl restart gunicorn

                                echo "✅ Deployment complete on EC2!"
                            '
                        """, returnStatus: true)

                        if (result != 0) {
                            error "❌ Remote deploy failed with exit code ${result}"
                        }
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

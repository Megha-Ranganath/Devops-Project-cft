pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "megharn/devops-project:latest"
        DEPLOY_FILE  = "deploy.yaml"
        DOMAIN       = "project-cicd.duckdns.org"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Megha-Ranganath/Devops-Project-cft.git'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh '''
                echo "📂 Current Directory:"
                pwd
                echo "📂 Files in Workspace:"
                ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "🔧 Building Docker image..."
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-id',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                    echo "🔐 Logging into Docker Hub..."
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                echo "📦 Pushing Docker image..."
                docker push $DOCKER_IMAGE
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "🚀 Deploying to Kubernetes..."
                microk8s.kubectl apply -f $DEPLOY_FILE
                sleep 15
                microk8s.kubectl get pods
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                echo "🌐 Verifying Application..."
                curl -I http://$DOMAIN || echo "⚠️ Application not reachable yet"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "❌ Build or Deployment failed. Check logs carefully."
        }
    }
}

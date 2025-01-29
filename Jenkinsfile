pipeline {
    agent any  // This defines where the pipeline will run (e.g., any available agent)
    
    environment {
        GIT_REPO = "https://github.com/your-username/your-repo.git"
        DOCKER_REGISTRY = "your-dockerhub-username"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'test', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_REGISTRY}/your-app:test .'
            }
        }

        stage('Push to Registry') {
            steps {
                sh 'docker push ${DOCKER_REGISTRY}/your-app:test'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}


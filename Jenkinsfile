pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "harishtiru"
        GIT_REPO = "https://github.com/harishtiru/app-web.git"
        KUBERNETES_NAMESPACE = "default"
    }
    stages {
        stage('Test') {
            steps {
                script {
                    runPipeline("test", null) // No feature branch for Test
                }
            }
        }
        stage('Development') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    runPipeline("development", "test")
                }
            }
        }
        stage('Staging') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    runPipeline("staging", "development")
                }
            }
        }
        stage('Production') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    runPipeline("production", "staging")
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution completed."
        }
    }
}

// Function to execute pipeline for each environment
def runPipeline(env, previousBranch) {
    def featureBranch = "feature-${env}-${UUID.randomUUID().toString().take(8)}"

    stage("Checkout Code (${env})") {
        checkout scm
        sh "git checkout ${env}"
    }

    if (env != "test") {
        stage("Create Feature Branch (${env})") {
            sh """
                git checkout -b ${featureBranch} ${previousBranch}
                git push origin ${featureBranch}
            """
        }
    }

    stage("Build Docker Image (${env})") {
        def imageTag = "${DOCKER_REGISTRY}/your-app:${env}-${UUID.randomUUID().toString().take(8)}"
        sh """
            docker build -t ${imageTag} .
            docker push ${imageTag}
        """
    }

    stage("Deploy to Kubernetes (${env})") {
        def imageTag = "${DOCKER_REGISTRY}/your-app:${env}"
        sh """
            kubectl set image deployment/your-deployment your-container=${imageTag} -n ${KUBERNETES_NAMESPACE}
        """
    }

    if (env != "test") {
        stage("Merge Feature Branch (${env})") {
            sh """
                git checkout ${env}
                git merge --no-ff ${featureBranch}
                git push origin ${env}
                git branch -d ${featureBranch}
                git push origin --delete ${featureBranch}
            """
        }
    }
}


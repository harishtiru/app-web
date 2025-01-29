pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "harishtiru"
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
        withCredentials([usernamePassword(credentialsId: 'gituserpass', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
            // Set Git credentials for cloning and checkout
            sh """
                rm -rf app-web
                git config --global user.name '${GIT_USERNAME}'
                git config --global user.password '${GIT_PASSWORD}'
                git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/harishtiru/app-web.git
                cd app-web
                git checkout ${env}
            """
        }
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
        def imageTag = "${DOCKER_REGISTRY}/test-app:${env}-${UUID.randomUUID().toString().take(8)}"
        sh """
            sudo docker build -t ${imageTag} .
            sudo docker push ${imageTag}
        """
    }

    stage("Deploy to Kubernetes (${env})") {
        def imageTag = "${DOCKER_REGISTRY}/test-app:${env}"
        sh """
            kubectl set image deployment/test-deployment test-container=${imageTag} -n ${KUBERNETES_NAMESPACE}
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


pipeline {
    agent any
    parameters {
        choice(name: 'BRANCH', choices: ['main', 'dev'], description: 'Select the branch to deploy')
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker image tag')
    }
    environment {
        BRANCH_NAME = "${params.BRANCH}"
        PORT = "${BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "*/${params.BRANCH}"]], userRemoteConfigs: [[url: '<https://github.com/AndreiMurillos/cicd-pipeline-jenkins']]])
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = BRANCH_NAME == 'main' ? "nodemain:${params.IMAGE_TAG}" : "nodedev:${params.IMAGE_TAG}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def imageName = BRANCH_NAME == 'main' ? "nodemain:${params.IMAGE_TAG}" : "nodedev:${params.IMAGE_TAG}"
                    sh """
                        docker stop ${BRANCH_NAME} || true
                        docker rm ${BRANCH_NAME} || true
                        docker run -d --expose ${PORT} -p ${PORT}:3000 --name ${BRANCH_NAME} ${imageName}
                    """
                }
            }
        }
    }
    post {
        always {
            script {
                sh "docker system prune -f"
            }
        }
    }
}
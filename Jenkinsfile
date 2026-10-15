
pipeline {
    agent any

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE_NAME = 'balaji00002/university-lms-portal'
        DEPLOYMENT = 'university-lms'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t %IMAGE_NAME%:v%BUILD_NUMBER% .'
            }
        }

        stage('Test Application') {
            steps {
                bat '"%DOCKER_EXE%" image inspect %IMAGE_NAME%:v%BUILD_NUMBER%'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat 'echo %DOCKER_PASSWORD% | "%DOCKER_EXE%" login -u %DOCKER_USERNAME% --password-stdin'
                    bat '"%DOCKER_EXE%" push %IMAGE_NAME%:v%BUILD_NUMBER%'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat '"%KUBECTL_EXE%" apply -f deployment.yaml'

                    bat '"%KUBECTL_EXE%" set image deployment/%DEPLOYMENT% university-lms=%IMAGE_NAME%:v%BUILD_NUMBER%'

                    try {
                        bat '"%KUBECTL_EXE%" rollout status deployment/%DEPLOYMENT% --timeout=90s'
                    } catch (err) {
                        echo 'Deployment health check failed. Rolling back to the previous release.'

                        bat '"%KUBECTL_EXE%" rollout undo deployment/%DEPLOYMENT%'
                        bat '"%KUBECTL_EXE%" rollout status deployment/%DEPLOYMENT% --timeout=90s'

                        error('New release failed. Rollback completed; inspect Kubernetes and the console log.')
                    }
                }
            }
        }

        stage('Verify Kubernetes') {
            steps {
                bat '"%KUBECTL_EXE%" get deployment university-lms'
                bat '"%KUBECTL_EXE%" get pods -l app=university-lms'
                bat '"%KUBECTL_EXE%" get service university-lms-service'
            }
        }
    }

    post {
        success {
            echo 'University LMS deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed. If rollout failed, check the rollback messages above.'
        }
    }
}

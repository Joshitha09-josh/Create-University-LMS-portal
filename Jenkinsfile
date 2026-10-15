
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'

        DOCKER_IMAGE = 'balaji00002/university-lms-portal'
        DEPLOYMENT_NAME = 'university-lms'
        CONTAINER_NAME = 'university-lms'
        SERVICE_NAME = 'university-lms-service'
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Cloning University LMS source from GitHub'
                checkout scm
                bat 'git log -1 --oneline'
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '"%KUBECTL_EXE%" config current-context'
                bat '"%KUBECTL_EXE%" get nodes'
                bat '"%KUBECTL_EXE%" get deployment'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build --pull -t %DOCKER_IMAGE%:v%BUILD_NUMBER% .'
            }
        }

        stage('Test Application') {
            steps {
                bat '"%DOCKER_EXE%" image inspect %DOCKER_IMAGE%:v%BUILD_NUMBER%'
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
                    bat '''
                        "%DOCKER_EXE%" login -u "%DOCKER_USERNAME%" --password-stdin <<< "%DOCKER_PASSWORD%"
                    '''
                    bat '"%DOCKER_EXE%" push %DOCKER_IMAGE%:v%BUILD_NUMBER%'
                }
            }
        }

        stage('Remember Stable Release') {
            steps {
                script {
                    def currentImage = bat(
                        script: '"%KUBECTL_EXE%" get deployment %DEPLOYMENT_NAME% -o jsonpath="{.spec.template.spec.containers[0].image}"',
                        returnStdout: true
                    ).trim()

                    if (!currentImage) {
                        error('Could not determine the current stable image. Check the deployment name and Kubernetes configuration.')
                    }

                    env.PREVIOUS_IMAGE = currentImage
                    echo "Previous image recorded: ${env.PREVIOUS_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '"%KUBECTL_EXE%" set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%DOCKER_IMAGE%:v%BUILD_NUMBER%'
            }
        }

        stage('Health Check') {
            steps {
                script {
                    try {
                        bat '"%KUBECTL_EXE%" rollout status deployment/%DEPLOYMENT_NAME% --timeout=120s'
                        bat '"%KUBECTL_EXE%" get deployment %DEPLOYMENT_NAME%'
                        bat '"%KUBECTL_EXE%" get pods -o wide'
                    } catch (err) {
                        echo 'New rollout failed. Restoring the previous image.'

                        bat '"%KUBECTL_EXE%" set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%PREVIOUS_IMAGE%'
                        bat '"%KUBECTL_EXE%" rollout status deployment/%DEPLOYMENT_NAME% --timeout=120s'

                        error('New release failed; rollback to the previous image was attempted.')
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '"%KUBECTL_EXE%" get deployment %DEPLOYMENT_NAME%'
                bat '"%KUBECTL_EXE%" get pods'
                bat '"%KUBECTL_EXE%" get service'
            }
        }
    }

    post {
        success {
            echo 'University LMS pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Console Output and rollback messages.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}


pipeline {
    agent any

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE_NAME = 'balaji00002/university-lms'
        IMAGE_TAG = 'v1'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Joshitha09-josh/University-Learning-Portal.git'
            }
        }

        stage('Check Project Files') {
            steps {
                bat 'if not exist index.html exit /b 1'
                bat 'if not exist Dockerfile exit /b 1'
                bat 'if not exist deployment.yaml exit /b 1'
                echo 'All project files are present.'
            }
        }

        stage('Check Docker and Kubernetes') {
            steps {
                bat '"%DOCKER_EXE%" --version'
                bat '"%DOCKER_EXE%" info'
                bat '"%KUBECTL_EXE%" version --client'
                bat '"%KUBECTL_EXE%" config current-context'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t %IMAGE_NAME%:%IMAGE_TAG% .'
                bat '"%DOCKER_EXE%" image inspect %IMAGE_NAME%:%IMAGE_TAG%'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal
                        echo %DOCKER_TOKEN% | "%DOCKER_EXE%" login --username "%DOCKER_USER%" --password-stdin
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push %IMAGE_NAME%:%IMAGE_TAG%
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {
                    bat '''
                        @echo off
                        set "KUBECONFIG=%KUBECONFIG_FILE%"
                        "%KUBECTL_EXE%" apply -f deployment.yaml
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" rollout status deployment/university-lms --timeout=120s
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" get pods
                        "%KUBECTL_EXE%" get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image pushed and Kubernetes deployment completed.'
        }
        failure {
            echo 'FAILED: Check Console Output for the first error.'
        }
        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

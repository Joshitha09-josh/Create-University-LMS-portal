
pipeline {
    agent any

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
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
                echo 'All required project files are present.'
            }
        }

        stage('Check Docker') {
            steps {
                bat '"%DOCKER_EXE%" --version'
                bat '"%DOCKER_EXE%" info'
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
                    withEnv(["KUBECONFIG=${KUBECONFIG_FILE}"]) {
                        bat 'kubectl apply -f deployment.yaml'
                        bat 'kubectl rollout status deployment/university-lms --timeout=120s'
                        bat 'kubectl get pods'
                        bat 'kubectl get services'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Image pushed and Kubernetes deployment completed.'
        }
        failure {
            echo 'FAILED: Review Console Output for the exact failing stage.'
        }
        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

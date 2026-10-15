
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        DOCKER_IMAGE = 'balaji00002/university-lms'
        IMAGE_TAG = 'v1'
    }

    stages {
        stage('Verify Project Files') {
            steps {
                bat '''
                    if not exist index.html exit /b 1
                    if not exist Dockerfile exit /b 1
                    if not exist deployment.yaml exit /b 1
                    echo All required files are present.
                '''
            }
        }

        stage('Check Docker') {
            steps {
                bat '''
                    "%DOCKER_EXE%" --version
                    if errorlevel 1 exit /b 1
                    "%DOCKER_EXE%" info
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '''
                    "%KUBECTL_EXE%" version --client
                    if errorlevel 1 exit /b 1
                    "%KUBECTL_EXE%" config current-context
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    "%DOCKER_EXE%" build -t "%DOCKER_IMAGE%:%IMAGE_TAG%" .
                    if errorlevel 1 exit /b 1
                    "%DOCKER_EXE%" image inspect "%DOCKER_IMAGE%:%IMAGE_TAG%"
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    bat '''
                        echo %DOCKERHUB_TOKEN% | "%DOCKER_EXE%" login --username "%DOCKERHUB_USER%" --password-stdin
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%:%IMAGE_TAG%"
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
                        set "KUBECONFIG=%KUBECONFIG_FILE%"
                        "%KUBECTL_EXE%" apply -f deployment.yaml
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" rollout status deployment/university-lms --timeout=120s
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {
                    bat '''
                        set "KUBECONFIG=%KUBECONFIG_FILE%"
                        "%KUBECTL_EXE%" get deployments
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" get pods
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" get services
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built, pushed, and deployed to Kubernetes.'
        }
        failure {
            echo 'FAILED: Check the first error in Console Output.'
        }
        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

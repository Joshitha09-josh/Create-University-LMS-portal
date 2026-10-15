
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE_NAME = 'balaji00002/university-lms'
        IMAGE_TAG = 'v1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm

                bat '''
                    echo Checking project files...
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
                    "%DOCKER%" --version
                    if errorlevel 1 exit /b 1

                    "%DOCKER%" info
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    "%DOCKER%" build -t %IMAGE_NAME%:%IMAGE_TAG% .
                    if errorlevel 1 exit /b 1

                    "%DOCKER%" image inspect %IMAGE_NAME%:%IMAGE_TAG%
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    bat '''
                        echo Logging in to Docker Hub...
                        "%DOCKER%" login -u "%DOCKERHUB_USERNAME%" --password-stdin < nul
                    '''
                    bat '''
                        echo %DOCKERHUB_PASSWORD% | "%DOCKER%" login -u "%DOCKERHUB_USERNAME%" --password-stdin
                        if errorlevel 1 exit /b 1

                        "%DOCKER%" push %IMAGE_NAME%:%IMAGE_TAG%
                        if errorlevel 1 exit /b 1

                        "%DOCKER%" logout
                    '''
                }
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '''
                    if not exist "%KUBECTL%" (
                        echo ERROR: kubectl.exe was not found.
                        exit /b 1
                    )

                    "%KUBECTL%" version --client
                    if errorlevel 1 exit /b 1
                '''

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-new',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {
                    bat '''
                        set "KUBECONFIG=%KUBECONFIG_FILE%"

                        "%KUBECTL%" config current-context
                        if errorlevel 1 exit /b 1

                        "%KUBECTL%" get nodes
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-new',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {
                    bat '''
                        set "KUBECONFIG=%KUBECONFIG_FILE%"

                        "%KUBECTL%" apply -f deployment.yaml
                        if errorlevel 1 exit /b 1

                        "%KUBECTL%" rollout status deployment/university-lms --timeout=120s
                        if errorlevel 1 exit /b 1

                        "%KUBECTL%" get deployments
                        "%KUBECTL%" get pods
                        "%KUBECTL%" get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image pushed and Kubernetes deployment completed.'
            echo 'Portal URL: http://localhost:30080'
        }

        failure {
            echo 'FAILED: Check the first failing stage in Console Output.'
        }

        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

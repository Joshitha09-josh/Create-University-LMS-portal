
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'balaji00002/university-lms'
        IMAGE_TAG  = 'v1'
        KUBECTL    = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
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
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                    if errorlevel 1 exit /b 1
                    docker image inspect %IMAGE_NAME%:%IMAGE_TAG%
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
                        echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin
                        if errorlevel 1 exit /b 1

                        docker push %IMAGE_NAME%:%IMAGE_TAG%
                        if errorlevel 1 exit /b 1

                        docker logout
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
            echo 'SUCCESS: Image built and pushed; Kubernetes deployment completed.'
            echo 'Check the service at http://localhost:30080'
        }

        failure {
            echo 'FAILED: Open Console Output and find the first failing stage.'
        }

        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

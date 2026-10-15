
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        DOCKER_IMAGE = 'balaji00002/university-lms:v1'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        KUBE_CREDENTIALS_ID = 'kubeconfig'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking project files...'
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

        stage('Build Docker Image') {
            steps {
                bat '''
                    "%DOCKER_EXE%" build -t "%DOCKER_IMAGE%" .
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Test Application') {
            steps {
                bat '''
                    "%DOCKER_EXE%" run --rm "%DOCKER_IMAGE%" nginx -t
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Push to Docker Hub') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASSWORD'
            )
        ]) {
            script {
                def secretFile = "${env.WORKSPACE}\\docker-password.tmp"
                try {
                    writeFile file: 'docker-password.tmp',
                              text: env.DOCKER_PASSWORD
                    bat '''
                        "%DOCKER_EXE%" login -u "%DOCKER_USER%" --password-stdin < docker-password.tmp
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%"
                        if errorlevel 1 exit /b 1
                    '''
                } finally {
                    bat 'if exist docker-password.tmp del /f /q docker-password.tmp'
                }
            }
        }
    }
}

        stage('Check Kubernetes') {
            steps {
                bat '''
                    "%KUBECTL_EXE%" version --client
                    if errorlevel 1 exit /b 1

                    "%KUBECTL_EXE%" get nodes
                    if errorlevel 1 exit /b 1

                    "%KUBECTL_EXE%" get deployments
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
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

    post {
        success {
            echo 'SUCCESS: University Learning Portal pipeline completed.'
            echo 'Open http://localhost:30080 to check the application.'
        }
        failure {
            echo 'FAILED: Open Console Output and check the first failed stage.'
        }
    }
}


pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        DOCKER_IMAGE = 'balaji00002/university-lms:v1'
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Check Required Files') {
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

        stage('Build Docker Image') {
            steps {
                bat '''
                    "%DOCKER_EXE%" build -t "%DOCKER_IMAGE%" .
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        "%DOCKER_EXE%" login -u "%DOCKER_USER%" --password-stdin < nul
                    '''
                    // Log in securely without putting the password in command arguments.
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:DOCKER_PASSWORD |
                            & $env:DOCKER_EXE login -u $env:DOCKER_USER --password-stdin
                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed"
                        }
                    '''
                    bat '''
                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%"
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Check Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:KUBECONFIG = $env:KUBECONFIG_FILE

                        & $env:KUBECTL_EXE config current-context
                        if ($LASTEXITCODE -ne 0) {
                            throw "Kubernetes context is unavailable"
                        }

                        & $env:KUBECTL_EXE get nodes
                        if ($LASTEXITCODE -ne 0) {
                            throw "Cannot connect to Kubernetes"
                        }
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
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:KUBECONFIG = $env:KUBECONFIG_FILE

                        & $env:KUBECTL_EXE apply -f deployment.yaml
                        if ($LASTEXITCODE -ne 0) {
                            throw "Kubernetes deployment failed"
                        }

                        & $env:KUBECTL_EXE rollout status deployment/university-lms --timeout=120s
                        if ($LASTEXITCODE -ne 0) {
                            throw "Deployment rollout failed"
                        }

                        & $env:KUBECTL_EXE get pods
                        & $env:KUBECTL_EXE get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: University Learning Portal pipeline completed.'
        }
        failure {
            echo 'FAILED: Open Console Output and inspect the first failing stage.'
        }
        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

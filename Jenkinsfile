
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

        stage('Check Files') {
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
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:DOCKER_PASSWORD |
                            & $env:DOCKER_EXE login `
                                --username $env:DOCKER_USER `
                                --password-stdin
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
                            throw "Kubernetes context check failed"
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
                            throw "Deployment apply failed"
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
            echo 'SUCCESS: University Learning Portal deployed.'
        }

        failure {
            echo 'FAILED: Check the first failing stage in Console Output.'
        }

        always {
            echo 'University Learning Portal pipeline finished.'
        }
    }
}

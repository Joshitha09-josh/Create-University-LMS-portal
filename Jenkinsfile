
pipeline {
    agent any

    options {
        timestamps()
    }

    parameters {
        booleanParam(
            name: 'SIMULATE_FAULTY_RELEASE',
            defaultValue: false,
            description: 'Use a deliberately unavailable image to test automatic rollback.'
        )
    }

    environment {
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE_REPO = 'balaji00002/university-lms'
    }

    stages {
        stage('Checkout and Verify Files') {
            steps {
                bat '''
                    if not exist index.html exit /b 1
                    if not exist Dockerfile exit /b 1
                    if not exist deployment.yaml exit /b 1
                    echo Required project files are present.
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
                    "%DOCKER_EXE%" build -t "%IMAGE_REPO%:%BUILD_NUMBER%" .
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                bat '''
                    "%DOCKER_EXE%" run --rm "%IMAGE_REPO%:%BUILD_NUMBER%" nginx -t
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
                    bat '''
                        @echo off
                        echo %DOCKER_PASSWORD% | "%DOCKER_EXE%" login -u "%DOCKER_USER%" --password-stdin
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push "%IMAGE_REPO%:%BUILD_NUMBER%"
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Check Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    bat '''
                        "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" config current-context
                        if errorlevel 1 exit /b 1
                        "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get nodes
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Deploy and Verify or Roll Back') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    script {
                        def stableImage = "${env.IMAGE_REPO}:${env.BUILD_NUMBER}"
                        def deploymentImage = params.SIMULATE_FAULTY_RELEASE
                            ? "${env.IMAGE_REPO}:faulty-release-does-not-exist"
                            : stableImage

                        bat '''
                            "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" apply -f deployment.yaml
                            if errorlevel 1 exit /b 1
                        '''

                        bat """
                            "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" set image deployment/university-lms lms=${deploymentImage}
                            if errorlevel 1 exit /b 1
                        """

                        try {
                            bat '''
                                "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" rollout status deployment/university-lms --timeout=90s
                                if errorlevel 1 exit /b 1
                            '''
                        } catch (err) {
                            echo 'ROLLOUT FAILED. Automatically restoring the previous ReplicaSet.'

                            bat '''
                                "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" rollout undo deployment/university-lms
                                if errorlevel 1 exit /b 1

                                "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" rollout status deployment/university-lms --timeout=120s
                                if errorlevel 1 exit /b 1

                                "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get pods
                                "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get services
                            '''

                            error('New release failed. Rollback completed; pipeline marked failed to record the failed release.')
                        }

                        bat '''
                            "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get deployments
                            "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get pods
                            "%KUBECTL_EXE%" --kubeconfig "%KUBECONFIG_FILE%" get services
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: The release was built, tested, published and deployed.'
        }
        failure {
            echo 'PIPELINE FAILED: Check the first error in Console Output. If deployment failed, inspect the rollback messages.'
        }
    }
}

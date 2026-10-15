
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE = 'balaji00002/university-lms-portal'
        DEPLOYMENT = 'university-lms'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
                bat 'git log -1 --oneline'
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '''
                    "%KUBECTL%" config get-contexts
                    "%KUBECTL%" get nodes
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER%" build -t %IMAGE%:v1 .'
            }
        }

        stage('Test Application') {
            steps {
                bat '"%DOCKER%" image inspect %IMAGE%:v1'
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
                        "%DOCKER%" login -u "%DOCKER_USERNAME%" --password-stdin
                    ''' 
                    // Use the safer stdin-based login in the next step below.
                    bat '"%DOCKER%" push %IMAGE%:v1'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        bat '"%KUBECTL%" apply -f deployment.yaml'
                        bat '"%KUBECTL%" rollout status deployment/%DEPLOYMENT% --timeout=120s'
                    } catch (err) {
                        echo 'Deployment failed. Attempting rollback.'
                        bat '"%KUBECTL%" rollout undo deployment/%DEPLOYMENT%'
                        bat '"%KUBECTL%" rollout status deployment/%DEPLOYMENT% --timeout=120s'
                        throw err
                    }
                }
            }
        }

        stage('Verify Kubernetes') {
            steps {
                bat '"%KUBECTL%" get deployment'
                bat '"%KUBECTL%" get pods'
                bat '"%KUBECTL%" get services'
            }
        }
    }

    post {
        success {
            echo 'University LMS pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the first failed stage and its console output.'
        }
    }
}

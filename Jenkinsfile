
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
                bat 'dir'
                bat 'if not exist index.html exit /b 1'
                bat 'if not exist Dockerfile exit /b 1'
                bat 'if not exist deployment.yaml exit /b 1'
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
            }
        }

        stage('Test Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" image inspect %IMAGE_NAME%:%IMAGE_TAG%'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: GitHub checkout, Docker checks, image build, and image validation completed.'
        }

        failure {
            echo 'FAILED: Check Console Output to identify the stage that failed.'
        }

        always {
            echo 'University Learning Portal pipeline execution finished.'
        }
    }
}

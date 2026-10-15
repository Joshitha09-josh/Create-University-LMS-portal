
pipeline {
    agent any

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
                bat 'docker --version'
                bat 'docker info'
            }
        }
    }

    post {
        success {
            echo 'Project files and Docker checks completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the Console Output for the exact error.'
        }
    }
}

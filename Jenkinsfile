pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'YOUR_DOCKERHUB_USERNAME/university-lms'
        DEPLOYMENT = 'university-lms'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    set -eu
                    docker rm -f lms-ci-test >/dev/null 2>&1 || true
                    docker run -d --name lms-ci-test -p 18080:80 $IMAGE_NAME:$BUILD_NUMBER
                    trap 'docker rm -f lms-ci-test >/dev/null 2>&1 || true' EXIT
                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:18080/ -o /tmp/lms.html; then
                            grep -q "University Learning Portal" /tmp/lms.html
                            exit $?
                        fi
                        sleep 2
                    done
                    exit 1
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        set -eu
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy and Rollback') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -eu
                        sed "s|IMAGE_PLACEHOLDER|$IMAGE_NAME:$BUILD_NUMBER|" deployment.yaml > /tmp/lms-deployment.yaml
                        kubectl apply -f /tmp/lms-deployment.yaml

                        if ! kubectl rollout status deployment/$DEPLOYMENT --timeout=120s; then
                            echo "Deployment failed; rolling back."
                            kubectl rollout undo deployment/$DEPLOYMENT
                            kubectl rollout status deployment/$DEPLOYMENT --timeout=120s
                            exit 1
                        fi

                        kubectl get deployment $DEPLOYMENT
                        kubectl get pods
                        kubectl get service university-lms-service
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'University LMS pipeline completed.'
        }
    }
}

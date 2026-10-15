
        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    bat '''
                        echo %DOCKERHUB_TOKEN% | "%DOCKER_EXE%" login --username "%DOCKERHUB_USER%" --password-stdin
                        if errorlevel 1 exit /b 1

                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%:%IMAGE_TAG%"
                        if errorlevel 1 exit /b 1

                        "%DOCKER_EXE%" logout
                    '''
                }
            }
        }

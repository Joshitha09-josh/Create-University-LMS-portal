
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

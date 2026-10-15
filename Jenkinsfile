stage('Deploy to Kubernetes') {
    steps {
        withCredentials([
            file(credentialsId: 'kubeconfig-new',
                 variable: 'KUBECONFIG_FILE')
        ]) {
            bat '''
                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" config get-contexts
                if errorlevel 1 exit /b 1

                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" get nodes
                if errorlevel 1 exit /b 1

                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" apply -f deployment.yaml
                if errorlevel 1 exit /b 1

                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" rollout status deployment/university-lms --timeout=120s
                if errorlevel 1 exit /b 1

                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" get pods
                if errorlevel 1 exit /b 1

                "C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" --kubeconfig="%KUBECONFIG_FILE%" get services
                if errorlevel 1 exit /b 1
            '''
        }
    }
}
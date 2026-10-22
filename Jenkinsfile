
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'priyadharshinib733/predictive-maintenance'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_GITHUB_USERNAME/fat2.git'
            }
        }

        stage('Parallel Validation') {
            parallel {
                stage('Check HTML') {
                    steps {
                        bat 'findstr /I /C:"Industrial Predictive Maintenance" index.html'
                    }
                }
                stage('Check Dockerfile and YAML') {
                    steps {
                        bat 'if not exist Dockerfile exit /b 1'
                        bat 'if not exist deployment.yaml exit /b 1'
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'index.html,Dockerfile,deployment.yaml,Jenkinsfile',
                                 fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kuberconfig',
                         variable: 'KUBECONFIG_FILE')
                ]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG_FILE%
                    kubectl apply -f deployment.yaml
                    kubectl rollout status deployment/predictive-maintenance --timeout=120s
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build, push and deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the stage console output.'
        }
    }
}

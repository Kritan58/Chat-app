pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "backend-app"
        FRONTEND_IMAGE = "frontend-app"
        BACKEND_DEPLOYMENT = 'backend-deployment'
        FRONTEND_DEPLOYMENT = 'frontend-deployment'
        KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-prod'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        timeout(time: 20, unit: 'MINUTES')
    }

    stages {
        stage('Cleanup & Checkout') {
            steps {
                deleteDir()
                sh 'docker system prune -f --filter "until=2h" || true'

                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [cleanBeforeCheckout()],
                    userRemoteConfigs: [[
                        credentialsId: 'Chat-app',
                        url: 'https://github.com/Kritan58/Chat-app.git'
                    ]]
                )
            }
        }

        stage('Build Images') {
            parallel {
                stage('Backend') {
                    steps {
                        sh '''
                            docker build -t ${BACKEND_IMAGE}:latest -f server/Dockerfile server/
                        '''
                    }
                }
                stage('Frontend') {
                    steps {
                        sh '''
                            docker build -t ${FRONTEND_IMAGE}:latest -f public/Dockerfile public/
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl apply -f k8s/backend-deployment.yaml
                        kubectl apply -f k8s/frontend-deployment.yaml
                        kubectl set image deployment/${BACKEND_DEPLOYMENT} backend=${BACKEND_IMAGE}:latest
                        kubectl set image deployment/${FRONTEND_DEPLOYMENT} frontend=${FRONTEND_IMAGE}:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -af --filter "until=1h" || true'
            deleteDir()
        }
    }
}

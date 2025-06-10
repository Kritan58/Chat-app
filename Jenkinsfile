pipeline {
    agent any
    
    environment {
        BACKEND_IMAGE = "backend-app"
        FRONTEND_IMAGE = "frontend-app"
        BACKEND_DEPLOYMENT = 'backend-deployment'
        FRONTEND_DEPLOYMENT = 'frontend-deployment'
        KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-prod'
        BUILD_TAG = "${BUILD_NUMBER}"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        timeout(time: 20, unit: 'MINUTES')
    }
    
    stages {
        stage('Cleanup & Checkout') {
            steps {
                // Clean workspace and old Docker images
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
                            docker rmi ${BACKEND_IMAGE}:latest || true
                            docker build -t ${BACKEND_IMAGE}:${BUILD_TAG} -f server/Dockerfile server/
                            docker tag ${BACKEND_IMAGE}:${BUILD_TAG} ${BACKEND_IMAGE}:latest
                        '''
                    }
                }
                stage('Frontend') {
                    steps {
                        sh '''
                            docker rmi ${FRONTEND_IMAGE}:latest || true
                            docker build -t ${FRONTEND_IMAGE}:${BUILD_TAG} -f public/Dockerfile public/
                            docker tag ${FRONTEND_IMAGE}:${BUILD_TAG} ${FRONTEND_IMAGE}:latest
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
                        kubectl set image deployment/${BACKEND_DEPLOYMENT} backend=${BACKEND_IMAGE}:${BUILD_TAG}
                        kubectl set image deployment/${FRONTEND_DEPLOYMENT} frontend=${FRONTEND_IMAGE}:${BUILD_TAG}
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

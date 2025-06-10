pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "backend-app"
    FRONTEND_IMAGE = "frontend-app"
    BACKEND_DEPLOYMENT = 'backend-deployment'
    FRONTEND_DEPLOYMENT = 'frontend-deployment'
    KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-prod'
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scmGit(
          branches: [[name: '*/main']],
          extensions: [],
          userRemoteConfigs: [[
            credentialsId: 'Chat-app',
            url: 'https://github.com/Kritan58/Chat-app.git'
          ]]
        )
        echo 'Code checkout successful'
      }
    }

    stage('Build Backend Image') {
      steps {
        sh '''
          docker build -t ${BACKEND_IMAGE}:latest -f server/Dockerfile server/
        '''
      }
    }

    stage('Build Frontend Image') {
      steps {
        sh '''
          docker build -t ${FRONTEND_IMAGE}:latest -f public/Dockerfile public/
        '''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo 'Deploying to Kubernetes cluster...'
        withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG_FILE

            kubectl apply -f k8s/backend-deployment.yaml --validate=false --insecure-skip-tls-verify=true 
            kubectl apply -f k8s/frontend-deployment.yaml --validate=false --insecure-skip-tls-verify=true

            kubectl set image deployment/${BACKEND_DEPLOYMENT} backend=${BACKEND_IMAGE}:latest --namespace=default
            kubectl set image deployment/${FRONTEND_DEPLOYMENT} frontend=${FRONTEND_IMAGE}:latest --namespace=default
          '''
        }
      }
    }
	   stage('Post Deployment stage') {
            steps {
                script {
                    sh 'docker system prune --all --force --filter "until=1h"'
                }
                echo 'Post deployment notification via SMTP server'
            }
        }
  }
}

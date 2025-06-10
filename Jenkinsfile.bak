pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "backend-app"
    FRONTEND_IMAGE = "frontend-app"
    BACKEND_DEPLOYMENT = 'backend-deployment'
    FRONTEND_DEPLOYMENT = 'frontend-deployment'
    KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-prod' // Jenkins kubeconfig secret
  }

  stages {
    stage('Gitcheckout SCM') {
            steps {
                // Checkout command
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Chat-app', url: 'https://github.com/Kritan58/Chat-app.git']])
                echo 'Code checkout successful'
            }
        }
 
    stage('Build Backend image') {
      steps {
        sh '''
        docker build -t ${BACKEND_IMAGE} -f server/Dockerfile server/
        '''
      }
    }


    stage('Build & Push Frontend Image') {
      steps {
          sh '''
            docker build -t ${FRONTEND_IMAGE} -f public/Dockerfile public/
          '''
        }
      }
    
 stage('Deploy Containers') {
   steps {
    script {
      // Stop & remove old containers if they exist (optional cleanup)
      sh '''
        docker rm -f backend-app || true
        docker rm -f frontend-app || true
      '''

      // Run backend container
      sh '''
        docker run -d --name backend-app -p 5000:5000 \
  -e PORT=5000 \
  -e MONGO_URL="mongodb://kritan:kritan@123@host.docker.internal:27017/kritanDb?authSource=admin" \
  backend-app

      '''

      // Run frontend container
      sh '''
        docker run -d --name frontend-app -p 3000:3000 frontend-app
      '''
    }
  }
}


    stage('Deploy to Kubernetes') {
  steps {
    echo 'Deploying to Kubernetes cluster...'

    // Apply Kubernetes manifests (you need these YAML files prepared)
    sh '''
    kubectl apply -f k8s/backend-deployment.yaml
    kubectl apply -f k8s/frontend-deployment.yaml
    '''

    // Set the image if you're rebuilding with new tags
    // Assuming `backend-app:latest` and `frontend-app:latest` are rebuilt locally
    sh '''
    kubectl set image deployment/${BACKEND_DEPLOYMENT} backend=${BACKEND_IMAGE}:latest --namespace=default
    kubectl set image deployment/${FRONTEND_DEPLOYMENT} frontend=${FRONTEND_IMAGE}:latest --namespace=default
    '''
  }
}

    }
  }


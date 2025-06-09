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
        docker build -t ${BACKEND_IMAGE} .
        '''
      }
    }


    stage('Build & Push Frontend Image') {
      steps {
        dir('public') {
          sh """
            docker build -t ${FRONTEND_IMAGE} .
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
        steps {
	echo 'Deployed to kubernetes'
       // withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
         // sh """
           // export KUBECONFIG=$KUBECONFIG

            //kubectl set image deployment/${BACKEND_DEPLOYMENT} backend=${BACKEND_IMAGE} --namespace=default
            //kubectl set image deployment/${FRONTEND_DEPLOYMENT} frontend=${FRONTEND_IMAGE} --namespace=default
         // """
        }
      }
    }
  }


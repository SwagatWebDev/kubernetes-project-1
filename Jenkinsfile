pipeline {

  options {
    ansiColor('xterm')
  }

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {

    stage('Kaniko Build & Push Image') {
      steps {
        container('kaniko') {
          script {
            sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                             --context `pwd` \
                             --destination=swagatkm/myweb:${BUILD_NUMBER}
            '''
          }
        }
      }
    }

    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
            sh 'mkdir -p $HOME/.kube'
            sh 'sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config'
            sh 'sudo chown $(id -u):$(id -g) $HOME/.kube/config'
            sh 'kubectl apply -f myweb.yaml'
          }
        }
      }
    }
  
  }
}

pipeline {
  agent {
    kubernetes {
      label 'jenkins-slaves'
      defaultContainer 'jnlp'
    }
  }
  stages {

    stage('Build Image') {
      steps {
        container('docker') {
          // Build new image
          sh "until docker ps; do sleep 3; done && docker build -t datahubid/argocd-demo:${env.GIT_COMMIT} ."
        }
      }
    }

    stage('Push Image') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      steps {
        container('docker') {
          // Publish new image
          sh "docker login -u $DOCKERHUB_CREDS_USR -p $DOCKERHUB_CREDS_PSW docker.io && docker push datahubid/argocd-demo:${env.GIT_COMMIT}"
        }
      }
    }

    stage('Deploy E2E') {
      environment {
        GIT_CREDS = credentials('git')
      }
      steps {
        container('tools') {
          sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/lukyr/argocd-demo-deploy.git"
          sh "git config --global user.email 'luky.ramdhanie@gmail.com'"

          dir("argocd-demo-deploy") {
            sh "cd ./e2e && kustomize edit set image datahubid/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }

    stage('Deploy to Prod') {
      steps {
        input message:'Approve deployment?'
        container('tools') {
          dir("argocd-demo-deploy") {
            sh "cd ./prod && kustomize edit set image datahubid/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }
  }
}

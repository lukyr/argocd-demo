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
          sh "until docker ps; do sleep 3; done && docker build -t 192.168.0.105:5000/argocd-demo:${env.GIT_COMMIT} ."
          sh "docker push 192.168.0.105:5000/argocd-demo:${env.GIT_COMMIT}"
        }
      }
    }

    stage('Push Image') {
      steps {
        container('docker') {
          // Push image
          sh "docker push 192.168.0.105:5000/argocd-demo:${env.GIT_COMMIT}"
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
          sh "git checkout local-registry"

          dir("argocd-demo-deploy") {
            sh "cd ./e2e && kustomize edit set image 192.168.0.105:5000/argocd-demo:${env.GIT_COMMIT}"
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
            sh "cd ./prod && kustomize edit set image 192.168.0.105:5000/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }
  }
}

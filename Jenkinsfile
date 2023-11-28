pipeline {

  agent {
    node {
      label 'jenkinsci-slave-isha-poc'
    }
   }
  options {
    buildDiscarder logRotator(
      daysToKeepStr: '7',
      numToKeepStr: '7'
      )
    }

  environment {
    BUILD_HASH = '${BUILD_NUMBER}'
    BUILD_HASH_ID = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).take(7)
    EMAIL_FROM = 'cicd.devops@ishafoundation.org'
    EMAIL_TO = 'senthil'
    DIST_ARCHIVE_VAR="dist.${JOB_NAME}-${BUILD_ID}.tar.gz"
    GIT_CREDS=credentials('github_cicd_user')
   }

  stages {

      stage ('Building Docker image') {
              steps {
                echo 'Building Docker image'
                 print 'BUILD_HASH_ID=' + BUILD_HASH_ID
                 sh 'docker image prune -af'
                 sh 'docker build -f Dockerfile-eks -t ishafoundationinc/k8s-dev-ishangam:${BUILD_HASH_ID} .'
              }
        }
      stage('Push Docker Image to Dockerhub') {
          steps {

            withCredentials([usernamePassword(credentialsId: 'dockerhub-ishacicddevops-user-id', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
            echo 'Login Dockerhub'
            echo ''' "$DOCKERHUB_PASS" | docker login -u $DOCKERHUB_USER --password-stdin https://index.docker.io/v1/ '''
            echo 'Pushing Docker image to Dockerhub'
            sh 'docker push ishafoundationinc/k8s-dev-ishangam:${BUILD_HASH_ID}'

            }
          }
        }

      stage('Checkout App Manifests on GitHub'){
        steps{
            git branch: 'argocd',
                credentialsId: 'github_cicd_user',
                url: 'https://github.com/ishadevopsbotsorg/cicd.git'
        }
      }

      stage('Check the contents of argocd manifest repo'){
        steps{
          script{
            dir("argocd/apps/dev/ishangam-web")
            {
              sh 'sed -i "s#ishafoundationinc/k8s-dev-ishangam:.*#ishafoundationinc/k8s-dev-ishangam:${BUILD_HASH_ID}#" dev-ishangam-web-deploy.yaml'
              sh 'cat dev-ishangam-web-deploy.yaml'
            }
          }
        }
      }

      stage('Git Push To argocd') {
        steps {
                sh 'ls'
                sh 'cd argocd'
                sh 'git add .'
                sh 'git commit -m "Image Tag Updated"'
                sh "git push https://${GIT_CREDS_USR}:${GIT_CREDS_PSW}@github.com/ishadevopsbotsorg/cicd.git"
            }
        }
    

      stage('Clean Workspace') {
            steps {
              cleanWs()
        }
      }
    }
    }

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
    DIST_ARCHIVE_VAR="dist.${JOB_NAME}-${BUILD_ID}.tar.gz"
    GIT_CREDS=credentials('github_cicd_user')
   }

  stages {

      stage ('Building Docker image') {
              steps {
                echo 'Building Docker image'
                 print 'BUILD_HASH_ID=' + BUILD_HASH_ID
                 sh 'docker image prune -af'
                 sh 'docker build -f Dockerfile -t rajitpaul/argocd-sample-app:${BUILD_HASH_ID} .'
              }
        }
      stage('Push Docker Image to Dockerhub') {
          steps {

            withCredentials([usernamePassword(credentialsId: 'rajit-dockerhub', usernameVariable: 'DOCKER_REGISTRY_USER', passwordVariable: 'DOCKER_REGISTRY_PWD')]) {
            echo 'Login Dockerhub'
            echo ''' "$DOCKER_REGISTRY_PWD | docker login -u $DOCKER_REGISTRY_USER --password-stdin '''
            echo 'Pushing Docker image to Dockerhub'
            sh 'docker push rajitpaul/argocd-sample-app:${BUILD_HASH_ID}'

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
            dir("argocd/apps/dev/sample-app")
            {
              sh 'sed -i "s#rajitpaul/argocd-sample-app:.*#rajitpaul/argocd-sample-app:${BUILD_HASH_ID}#" deploy.yaml'
              sh 'cat deploy.yaml'
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

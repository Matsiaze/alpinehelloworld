/* Import shared Library */
@Library('Matsiaze-shared-library') _

pipeline {
    environment {
      IMAGE_NAME = 'myalpine-jenkins'
      IMAGE_TAG = 'latest'
      STAGING = 'myalpine-staging'
      PRODUCTION = 'myalpine-production'
    }
    agent none
    stages {
        stage('Build Alpine Image') {
            agent any
            steps {
                sh 'docker build -t mclab7/${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }
        stage('Run Container') {
            agent any
            steps {
               script {
                 sh '''
                   docker rm -f ${IMAGE_NAME}
                   docker run -d --name ${IMAGE_NAME} -e PORT=5000 -p 5000:5000 mclab7/${IMAGE_NAME}:${IMAGE_TAG}
                   sleep 7
                 '''
               }
            }
        }
        stage('Test Image') {
            agent any
            steps {
                sh 'curl http://192.168.56.9:5000 | grep -q -i "hello world"'
            }
        }
        stage('Clean Container') {
            agent any
            steps {
               script {
                 sh '''
                   docker stop ${IMAGE_NAME}
                   docker rm ${IMAGE_NAME}
                 '''
               }
            }
        }
        stage('Push and deploy to staging') {
          when {
                 expression { GIT_BRANCH = 'origin/main' }
               }
            agent any
            environment {
              HEROKU_API_KEY = credentials ('HEROKU_API_KEY')
            }
            steps {
              script {
                 sh '''
                   heroku container:login
                   heroku create $STAGING || echo 'project already exist'
                   heroku container:push -a $STAGING web
                   heroku container:release -a $STAGING web
                 '''
               }
            }
        }
        stage('Push and deploy to production') {
          when {
                 expression { GIT_BRANCH = 'origin/main'}
               }
            agent any
            environment {
              HEROKU_API_KEY = credentials ('HEROKU_API_KEY')
            }
            steps {
              script {
                 sh '''
                   heroku container:login
                   heroku create $PRODUCTION || echo 'project already exist'
                   heroku container:push -a $PRODUCTION web
                   heroku container:release -a $PRODUCTION web
                 '''
               }
            }
        }
    }
  post {
    always {
      script {
	slackNotifier currentBuild.result
      }
    }
  }
}

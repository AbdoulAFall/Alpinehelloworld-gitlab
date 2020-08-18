def IMAGE_NAME = "alpinehelloworld"
def IMAGE_TAG = "latest"
def STAGING = "eazytraining-staging"
def PRODUCTION = "eazytraining-production"
pipeline {
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
              script {
                sh 'docker build -t eazytraining/$IMAGE_NAME:$IMAGE_TAG .'
              }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
              script {
                docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
                sleep 5
              }
            }
        }
        stage('Test image') {
            agent any
            steps {
              script {
                curl http://localhost | grep -q "Hello world!"
              }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                     expression { GIT_BRANCH == 'origin/master' }
                 }
            agent any
            environment {
              HEROKY_API_KEY = credentials('heroku_api_key')
            }
            steps {
              script {
                heroku container:login
                heroku create $STAGING || echo "project already exist"
                heroku container:push -a $STAGING web
                heroku container:release -a $STAGING web
              }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                     expression { GIT_BRANCH == 'origin/master' }
                 }
            agent any
            environment {
              HEROKY_API_KEY = credentials('heroku_api_key')
            }
            steps {
              script {
                heroku container:login
                heroku create $STAGING || echo "project already exist"
                heroku container:push -a $PRODUCTION web
                heroku container:release -a $PRODUCTION web
              }
            }
        }
    }
}

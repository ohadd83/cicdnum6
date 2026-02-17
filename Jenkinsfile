pipeline {
    agent any

    environment {
        APP_NAME = "myappnew1"
        IMAGE_TAG = "${GIT_COMMIT.take(7)}"   // short Git SHA for versioning
        DEV_CONTAINER = "myapp-dev"
        STAGING_CONTAINER = "myapp-staging"
        PROD_CONTAINER = "myapp-prod"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10')) // keep last 10 builds
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint || echo "Lint warnings"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $APP_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker tag $APP_NAME:$IMAGE_TAG $USER/$APP_NAME:$IMAGE_TAG
                        docker push $USER/$APP_NAME:$IMAGE_TAG
                    '''
                }
            }
        }


        stage('Approval for STAGING') {
            steps {
                input message: "Deploy to STAGING?", ok: "Yes"
            }
        }

        stage('Deploy to STAGING') {
            steps {
                sh '''
                    docker stop $STAGING_CONTAINER || true
                    docker rm $STAGING_CONTAINER || true
                    docker run -d -p 8081:3000 --name $STAGING_CONTAINER $APP_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval for PROD') {
            steps {
                input message: "Deploy to PROD?", ok: "Yes"
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh '''
                    docker stop $PROD_CONTAINER || true
                    docker rm $PROD_CONTAINER || true
                    docker run -d -p 8087:3000 --name $PROD_CONTAINER $APP_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo "✅ Pipeline completed successfully!"
            // Optional: send email or Slack notification
        }

        failure {
            echo "❌ Pipeline failed!"
            // Optional: send failure notification
        }
    }
}


pipeline {
    agent any
    options { timestamps(); disableConcurrentBuilds() }
    environment {
        IMAGE_NAME = 'shopwave-ecommerce'
        CONTAINER_NAME = 'shopwave-web'
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Validate') {
            steps {
                sh 'test -f index.html && test -f styles.css && test -f app.js && test -f Dockerfile'
                sh 'echo "Application files validated"'
            }
        }
        stage('Build Docker image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true
                    docker run -d --restart unless-stopped --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
        stage('Smoke test') {
            steps {
                sh '''
                    for attempt in $(seq 1 10); do
                      if curl --fail --silent http://127.0.0.1/ > /dev/null; then
                        echo "ShopWave is responding on port 80."
                        exit 0
                      fi
                      sleep 3
                    done
                    docker logs ${CONTAINER_NAME} || true
                    exit 1
                '''
            }
        }
    }
    post {
        success { echo 'Deployment successful: ShopWave is live.' }
        failure { echo 'Deployment failed. Review stage logs and Docker logs.' }
    }
}

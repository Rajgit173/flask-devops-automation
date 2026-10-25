pipeline {
    agent any

    environment {
        APP_NAME = 'flask-devops-automation'
        IMAGE_NAME = 'flask-devops-automation'
        APP_PORT = '8082'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER} \
                      python -m unittest discover -s tests -v
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME} || true

                    docker run -d \
                      --name ${APP_NAME} \
                      --restart unless-stopped \
                      -p ${APP_PORT}:5000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 15); do
                      if curl -fsS http://127.0.0.1:${APP_PORT}/health; then
                        echo "Application is healthy"
                        exit 0
                      fi
                      sleep 2
                    done

                    echo "Application health check failed"
                    docker logs ${APP_NAME} || true
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Flask DevOps Automation pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Review the Jenkins console output.'
        }
    }
}

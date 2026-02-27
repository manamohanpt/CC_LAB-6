pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."

                # Create network if not exists
                docker network create app-network || true

                # Remove old containers if any
                docker rm -f backend1 backend2 || true

                # Run two backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for containers to fully start
                sleep 3
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying NGINX load balancer..."

                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Run nginx container
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # Wait for nginx to initialize
                sleep 3

                # Copy configuration file
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                sleep 2

                # Reload nginx safely
                docker exec nginx-lb nginx -s reload || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}

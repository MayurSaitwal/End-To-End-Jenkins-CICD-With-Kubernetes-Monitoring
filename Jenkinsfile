pipeline {
    agent { label 'cicd' }

    stages {

        stage('Code') {
            steps {
                echo 'This is the cloning of the code'

                git branch: 'main',
                    url: 'https://github.com/MayurSaitwal/Autonomous-Restaurant-Management-System.git'

                echo 'Code cloned successfully'
            }
        }
        stage('Check Docker Access') {
    steps {
        sh '''
            echo "===== USER ====="
            whoami
            echo "===== ID ====="
            id
            echo "===== DOCKER ====="
            docker --version
            docker ps
        '''
    }
}
        stage('Build') {
            steps {
                echo 'This is the building of code'

                sh '''
                    docker build --no-cache -t restaurant-image:latest .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'This is the testing phase'

                sh '''
                    python3 -m py_compile app.py config.py models.py
                '''
            }
        }

        stage('Deploy with Docker') {
            steps {
                echo 'Deploying application using Docker'

                sh '''
                    docker stop res-app || true
                    docker rm res-app || true

                    docker run -d \
                        --name res-app \
                        -p 5000:5000 \
                        restaurant-image:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker tag restaurant-image:latest \
                            $DOCKER_USERNAME/restaurant-image:latest

                        docker push \
                            $DOCKER_USERNAME/restaurant-image:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                echo 'This is the deployment stage of Kubernetes'

                sh '''
                    kubectl apply -f k8s/namespace.yml

                    kubectl apply -f k8s/pvc.yml
                    kubectl apply -f k8s/mysql-secret.yml
                    kubectl apply -f k8s/mysql-deployment.yml
                    kubectl apply -f k8s/mysql-service.yml
                    kubectl apply -f k8s/flask-deployment.yml
                    kubectl apply -f k8s/flask-service.yml

                    kubectl rollout restart deployment/flask-app -n restaurant

                    kubectl rollout status deployment/flask-app -n restaurant
                '''
            }
        }

        stage('Verify Deployment') {
    steps {
        echo 'Checking Kubernetes deployment'

        sh 'kubectl rollout status deployment/mysql-deployment -n restaurant --timeout=120s'
        sh 'kubectl rollout status deployment/flask-app -n restaurant --timeout=120s'

        sh 'kubectl get deployments -n restaurant'
        sh 'kubectl get pods -n restaurant'
        sh 'kubectl get services -n restaurant'
    }
}
    }
}

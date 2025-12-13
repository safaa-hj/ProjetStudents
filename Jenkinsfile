pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'safahajji123/spring-app'
        DOCKER_TAG = 'latest'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                echo 'Checking out code from Git...'
                checkout scm
            }
        }
        
        stage('Clean & Compile') {
            steps {
                echo 'Nettoyage et compilation du projet'
                sh 'mvn clean compile'
            }
        }

        /*stage('SonarQube Analysis') {
            steps {
                echo 'Analyse de la qualité du code avec SonarQube'
                sh '''
                    mvn sonar:sonar \
                    -Dsonar.host.url=http://192.168.50.4:9000 \
                    -Dsonar.login=safa \
                    -Dsonar.password=Safahajji123+
                '''
            }
        }*/

        stage('SonarQube Analysis') {
            steps {
                echo 'Analyse de la qualité du code avec SonarQube'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.host.url=http://192.168.50.4:9000 \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }


        stage('Build with Maven') {
            steps {
                echo 'Building with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."

                    echo 'Pushing Docker image...'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    sh '''
                        echo "Checking kubectl connectivity..."
                        kubectl cluster-info || echo "Warning: Could not reach cluster"
                        kubectl get nodes || echo "Warning: Could not list nodes"
                        
                        echo "Applying MySQL deployment..."
                        kubectl apply -f kubernetes/mysql-deployment.yaml -n devops --validate=false || {
                            echo "Failed to apply mysql-deployment.yaml"
                            exit 1
                        }
                        
                        echo "Applying Spring deployment..."
                        kubectl apply -f kubernetes/spring-deployment.yaml -n devops --validate=false || {
                            echo "Failed to apply spring-deployment.yaml"
                            exit 1
                        }
                        
                        echo "Restarting Spring app deployment..."
                        kubectl rollout restart deployment spring-app -n devops || echo "Warning: Could not restart deployment (may not exist yet)"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh 'kubectl get pods -n devops'
                sh 'kubectl get svc -n devops'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
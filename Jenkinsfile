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
        
/*
stage('Deploy to Kubernetes') {
    steps {
        echo 'Deploying to Kubernetes...'
        sh 'kubectl apply -f mysql-deployment.yaml -n devops'
        sh 'kubectl apply -f spring-deployment.yaml -n devops'
        sh 'kubectl rollout restart deployment spring-app -n devops'
    }
}


        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh 'kubectl get pods -n devops'
                sh 'kubectl get svc -n devops'
            }
        }
           */
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

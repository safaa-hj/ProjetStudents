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
                    // Option 1: Use kubeconfig from Jenkins credentials (recommended)
                    // First, add your kubeconfig as a "Secret file" credential in Jenkins
                    // with ID 'kubeconfig-file', then uncomment the following:
                    /*
                    withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                        sh '''
                            export KUBECONFIG=${KUBECONFIG}
                            kubectl config view --minify
                            kubectl cluster-info
                            kubectl get nodes
                        '''
                    }
                    */
                    
                    // Option 2: Use kubeconfig content from secret text credential
                    // Add your kubeconfig content as a "Secret text" credential with ID 'kubeconfig-content'
                    withCredentials([string(credentialsId: 'kubeconfig-content', variable: 'KUBECONFIG_CONTENT')]) {
                        sh '''
                            echo "=== Setting up kubectl configuration ==="
                            
                            # Create .kube directory if it doesn't exist
                            mkdir -p ~/.kube
                            
                            # Write kubeconfig content to file
                            echo "$KUBECONFIG_CONTENT" > ~/.kube/config
                            chmod 600 ~/.kube/config
                            
                            echo "=== Kubectl Configuration Diagnostics ==="
                            echo "Checking kubectl version..."
                            kubectl version --client || {
                                echo "ERROR: kubectl not found or not working"
                                exit 1
                            }
                            
                            echo "Checking current context..."
                            kubectl config current-context || {
                                echo "ERROR: No context set in kubeconfig"
                                exit 1
                            }
                            
                            echo "Checking kubeconfig..."
                            kubectl config view --minify || {
                                echo "ERROR: Could not view kubeconfig"
                                exit 1
                            }
                            
                            echo "Testing API server connectivity..."
                            if ! kubectl cluster-info 2>&1 | grep -q "Kubernetes control plane"; then
                                echo "WARNING: Could not reach Kubernetes API server"
                                echo "Attempting to continue with --validate=false..."
                            fi
                            
                            echo "Checking if namespace exists..."
                            if ! kubectl get namespace devops 2>/dev/null; then
                                echo "Creating devops namespace..."
                                kubectl create namespace devops || {
                                    echo "ERROR: Failed to create namespace"
                                    exit 1
                                }
                            fi
                            
                            echo "=== Applying Deployments ==="
                            echo "Applying MySQL deployment..."
                            kubectl apply -f kubernetes/mysql-deployment.yaml -n devops --validate=false || {
                                echo "ERROR: Failed to apply mysql-deployment.yaml"
                                exit 1
                            }
                            
                            echo "Applying Spring deployment..."
                            kubectl apply -f kubernetes/spring-deployment.yaml -n devops --validate=false || {
                                echo "ERROR: Failed to apply spring-deployment.yaml"
                                exit 1
                            }
                            
                            echo "Restarting Spring app deployment..."
                            kubectl rollout restart deployment spring-app -n devops || echo "INFO: Could not restart deployment (may not exist yet, will be created)"
                        '''
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    withCredentials([string(credentialsId: 'kubeconfig-content', variable: 'KUBECONFIG_CONTENT')]) {
                        sh '''
                            export KUBECONFIG=~/.kube/config
                            echo "=== Deployment Status ==="
                            kubectl get pods -n devops || echo "Could not list pods"
                            kubectl get svc -n devops || echo "Could not list services"
                            kubectl get deployments -n devops || echo "Could not list deployments"
                        '''
                    }
                }
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
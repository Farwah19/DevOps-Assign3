pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-web-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Code Linting') {
            steps {
                script {
                    echo '========== Stage 1: Code Linting =========='
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install pylint
                        pylint app.py --disable=C0111,C0103,W0703 || true
                    '''
                }
            }
        }
        
        stage('Code Build') {
            steps {
                script {
                    echo '========== Stage 2: Building Docker Image =========='
                    sh '''
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
        
        stage('Unit Testing') {
            steps {
                script {
                    echo '========== Stage 3: Running Unit Tests =========='
                    sh '''
                        . venv/bin/activate
                        pip install -r requirements.txt
                        pytest tests/test_app.py -v
                    '''
                }
            }
        }
        
        stage('Containerized Deployment') {
            steps {
                script {
                    echo '========== Stage 4: Deploying Containers =========='
                    sh '''
                        # Stop and remove all compose resources
                        docker-compose down -v || true
                        
                        # Start new containers
                        docker-compose up -d
                        
                        # Wait for services to be healthy
                        sleep 20
                        
                        # Check if services are running
                        docker-compose ps
                        
                        # Show logs
                        docker-compose logs --tail=50
                    '''
                }
            }
        }
        
        
        stage('Selenium Testing') {
            steps {
                script {
                    echo '========== Stage 5: Running Selenium Tests =========='
                    sh '''
                        # Build Selenium test container
                        docker build -f Dockerfile.selenium -t selenium-tests .
                        
                        # Run Selenium tests in Docker network
                        docker run --network my-web-app_default \
                            --name selenium-tests-${BUILD_NUMBER} \
                            selenium-tests || true
                        
                        # Cleanup
                        docker rm selenium-tests-${BUILD_NUMBER} || true
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo '========== Pipeline Completed =========='
            sh 'docker-compose logs app'
        }
        success {
            echo '✓ Pipeline executed successfully!'
        }
        failure {
            echo '✗ Pipeline failed. Check logs above.'
            sh 'docker-compose down || true'
        }
    }
}

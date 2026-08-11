pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        booleanParam(name: 'PUSH_IMAGES', defaultValue: true, description: 'Push images to Docker Hub and AWS ECR')
        booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploy with Docker Compose')
    }

    environment {
        AWS_REGION = 'ap-south-1'
        DOCKERHUB_BACKEND = "${env.DOCKERHUB_USERNAME}/travel-booking-backend"
        DOCKERHUB_FRONTEND = "${env.DOCKERHUB_USERNAME}/travel-booking-frontend"
        ECR_BACKEND = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/travel-booking-backend"
        ECR_FRONTEND = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/travel-booking-frontend"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Backend') {
            steps {
                  sh '''
            export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
            export PATH=$JAVA_HOME/bin:$PATH

            java -version
            cd api
            ./gradlew clean bootJar -x test
        '''
            }
        }

        stage('Test Backend') {
            steps {
                sh '''
            export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
            export PATH=$JAVA_HOME/bin:$PATH

            java -version
            cd api
            ./gradlew test
        '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    cd public/booking-app
                    npm install
                    npm run build
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner                           -Dsonar.projectKey=travel-booking                           -Dsonar.projectName="Travel Booking System"                           -Dsonar.sources=api/src,public/booking-app/src                           -Dsonar.exclusions="**/node_modules/**,**/build/**"
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -f Dockerfile.backend -t ${DOCKERHUB_BACKEND}:${IMAGE_TAG} .
                    docker build -f Dockerfile.frontend -t ${DOCKERHUB_FRONTEND}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Images') {
            when { expression { params.PUSH_IMAGES } }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
                        docker push ${DOCKERHUB_BACKEND}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_FRONTEND}:${IMAGE_TAG}

                        aws ecr get-login-password --region ${AWS_REGION} |
                          docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                        docker tag ${DOCKERHUB_BACKEND}:${IMAGE_TAG} ${ECR_BACKEND}:${IMAGE_TAG}
                        docker tag ${DOCKERHUB_FRONTEND}:${IMAGE_TAG} ${ECR_FRONTEND}:${IMAGE_TAG}

                        docker push ${ECR_BACKEND}:${IMAGE_TAG}
                        docker push ${ECR_FRONTEND}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy') {
            when { expression { params.DEPLOY } }
            steps {
                sh '''
                    echo "Deploying ${IMAGE_TAG} to ${ENVIRONMENT}"
                    docker compose pull
                    docker compose up -d
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'api/build/libs/*.jar', allowEmptyArchive: true
            junit testResults: 'api/build/test-results/test/*.xml', allowEmptyResults: true
        }
    }
}

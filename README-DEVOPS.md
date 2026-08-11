# Travel Booking System - DevOps Layer

Application source:
https://github.com/Davidsksilva/booking-app

The upstream application is a travel booking system with React frontend, Spring Boot backend and MySQL database. This DevOps layer adds containerization and CI/CD automation.

## DevOps stack

GitHub | Jenkins | GitHub Actions | SonarQube | Docker | Docker Compose | Docker Hub | AWS ECR | AWS EC2

## Pipeline

GitHub -> Build/Test -> SonarQube -> Quality Gate -> Docker Build -> Docker Hub/AWS ECR -> EC2 -> Docker Compose

## Added files

- Dockerfile.backend
- Dockerfile.frontend
- docker-compose.yml
- database/init.sql
- Jenkinsfile
- .github/workflows/ci-cd.yml
- sonar-project.properties
- .env.example

## Local deployment

After copying these files into the application repository:

    docker compose build
    docker compose up -d
    docker compose ps

Frontend: http://localhost:3000
Backend: http://localhost:7070

Stop:

    docker compose down

## Jenkins requirements

Jenkins agent should have Docker, AWS CLI, Java 11, Node/npm and SonarScanner CLI.

Configure:
- Docker Hub credential ID: dockerhub-credentials
- SonarQube installation name: SonarQube
- AWS credentials or an EC2/IAM role with ECR push permission
- AWS_ACCOUNT_ID and DOCKERHUB_USERNAME as Jenkins environment/credential values

The Jenkins Quality Gate uses `abortPipeline: true`, so a failed gate stops the pipeline before image publishing/deployment.

## GitHub Actions secrets

- SONAR_TOKEN
- SONAR_HOST_URL
- DOCKERHUB_USERNAME
- DOCKERHUB_TOKEN

## Attribution

Keep the upstream project's license and attribution requirements when publishing a fork or derivative repository.

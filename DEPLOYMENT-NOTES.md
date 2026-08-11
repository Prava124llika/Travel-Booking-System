# Deployment Notes

The DevOps files are tailored to the selected repository structure:

- Backend: api/
- Frontend: public/booking-app/
- Backend port: 7070
- Database: MySQL
- Java: 11
- Gradle: 4.x

Important: the upstream application was designed for local development and also references separate Flight and Hotel backends. The Docker setup provided here is the DevOps foundation; some application/API configuration may need adjustment during integration.

Never commit passwords, AWS keys, Docker Hub tokens or SonarQube tokens. Use Jenkins Credentials, GitHub Secrets and AWS IAM roles/OIDC.

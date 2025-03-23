pipeline {
    agent any

    environment {
        // Environment variables for Nexus, SonarQube, and AWS ECR
        SONARQUBE_URL = 'http://sonarqube.example.com'
        SONARQUBE_TOKEN = credentials('sonarqube-token')
        NEXUS_REPO_URL = 'http://nexus.example.com/repository/maven-releases/'
        AWS_ACCOUNT_ID = '123456789012'
        REGION = 'us-east-1'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from the GitHub repository
                git branch: 'main', url: 'https://github.com/your-repo/petclinic-microservices.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                // Run OWASP Dependency Check
                script {
                    sh '''
                        mvn org.owasp:dependency-check-maven:check
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.host.url=${SONARQUBE_URL} \
                        -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Publish JAR to Nexus') {
            steps {
                script {
                    // Publish the artifact to Nexus Repository
                    sh """
                        mvn clean package deploy \
                        -DaltDeploymentRepository=nexus-repo::default::${NEXUS_REPO_URL}
                    """
                }
            }
        }

        stage('Build Docker Images') {
            matrix {
                axes {
                    axis {
                        name 'MICROSERVICE'
                        values 'customers-service', 'vets-service', 'visits-service'
                    }
                }
                stages {
                    stage('Build Docker Image') {
                        steps {
                            script {
                                // Build Docker image for each microservice
                                sh """
                                    docker build -t ${ECR_REGISTRY}/petclinic-${MICROSERVICE}:latest -f ${MICROSERVICE}/Dockerfile ${MICROSERVICE}
                                """
                            }
                        }
                    }
                    stage('Trivy Security Scan') {
                        steps {
                            script {
                                // Run Trivy scan for vulnerabilities
                                sh """
                                    trivy image --exit-code 1 --severity HIGH,CRITICAL ${ECR_REGISTRY}/petclinic-${MICROSERVICE}:latest || exit 1
                                """
                            }
                        }
                    }
                    stage('Push Docker Image to ECR') {
                        steps {
                            script {
                                // Push the Docker image to ECR if no vulnerabilities are found
                                sh """
                                    docker push ${ECR_REGISTRY}/petclinic-${MICROSERVICE}:latest
                                """
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for more details.'
        }
    }
}

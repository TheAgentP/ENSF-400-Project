pipeline{
    agent any

    // Set environment variables for the image
    environment {
        IMAGE_NAME = 'ensf400-project'
        TAG = 'latest'
        CREDENTIALS_ID = 'docker-pat'
        DOCKER_USER = 'theagentp'
        DOCKER_PASS = 'GreenBean4066!'
    }

    stages{
        // Stage for building the image
        stage('Build') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t $IMAGE_NAME:$TAG .'
                }
            }
        }

        // Stage for pushing the image to DockerHub
        stage('Push to DockerHub') {
            steps {
                script {
                    // Use DockerHub credentials (the ID you gave it in Jenkins)
                    withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Log in to DockerHub using the credentials
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker build -t $DOCKER_USER/$IMAGE_NAME:$TAG .
                            docker push $DOCKER_USER/$IMAGE_NAME:$TAG
                        '''
                    }
                }
            }
        }


        //Running the tests:
        stage('Unit Tests') {
            agent {
                docker {
                    image 'gradle:7.6.1-jdk11'
                }
            }
            steps {
                sh './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        // Security Analysis with OWASP's "DependencyCheck"
        stage('Dependency Check') {
            agent {
                docker {
                    image 'gradle:7.6.1-jdk11'
                }
            }
            steps {
                script {
                    // Run OWASP Dependency-Check for security analysis
                    sh './gradlew dependencyCheckAnalyze'
                }
            }
        }

        //Generate and save JavaDocs as an artifact
        stage('Generate JavaDocs') {
            agent {
                docker {
                    image 'gradle:7.6.1-jdk11'
                }
            }
            steps {
                // Generate JavaDocs
                sh './gradlew javadoc'
                // Archive the generated JavaDocs as build artifacts
                archiveArtifacts allowEmptyArchive: true, artifacts: 'build/docs/javadoc/**'
            }
        }

        // Stage for pulling the image and running the application
        stage('Deploy Application') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker pull $DOCKER_USER/$IMAGE_NAME:$TAG
                            docker run -di -p 8081:8080 $DOCKER_USER/$IMAGE_NAME:$TAG
                        '''
                    }
                }
            }
        }

        // Use a docker in docker container to run sonarcube (figured it out)
        stage('Static Analysis') {
            steps {
                script {
                    // Start SonarQube Server
                    sh 'docker run -d --name sonarqube -p 9000:9000 --pull always sonarqube:9.2-community'
                    sh 'echo "Waiting for SonarQube to start..." && sleep 60'
                    sh 'curl -X POST "http://host.docker.internal:9000/api/users/change_password" -H "Content-Type: application/x-www-form-urlencoded" -d "login=admin&previousPassword=admin&password=password" -u admin:admin'

                    // Build the SonarQube analysis image
                    sh 'docker build -t sonarqube-analysis -f Dockerfile.sonarqube-analysis .'

                    // Run the analysis
                    sh 'docker run sonarqube-analysis'
                }
            }
        }
    }
    post {
        always {
            // Clean up unused Docker resources (optional)
            sh 'docker system prune -f || true'
            sh 'docker stop sonarqube && docker rm sonarqube || true'
            sh 'docker rm sonarqube-analysis'
        }
    }
}

pipeline{
    agent any

    // Set environment variables for the image
    environment {
        IMAGE_NAME = 'ensf400-project'
        TAG = 'latest'
        DOCKER_USER = 'sslaquerre07'
        DOCKER_PASS = 'Pucky1120!'
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
                    withCredentials([usernamePassword(credentialsId: '59a71b5b-9cc7-4d75-b0ba-449b52a4ee27', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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
                    withCredentials([usernamePassword(credentialsId: '59a71b5b-9cc7-4d75-b0ba-449b52a4ee27', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker pull $DOCKER_USER/$IMAGE_NAME:$TAG
                            docker run -di -p 8081:8080 $DOCKER_USER/$IMAGE_NAME:$TAG
                        '''
                    }
                }
            }
        }

        //Use a docker in docker container to run sonarcube (can't figure out)
        stage('Static Analysis') {
            agent {
                docker {
                    image 'docker:20.10.7-dind'
                    args '--privileged'
                }
            }
            environment {
                SONAR_HOST_URL = 'http://localhost:9000'
                SONAR_TOKEN = credentials('your-sonar-token-id')  // Jenkins credentials
            }
            steps {
                script {
                    sh '''
                        docker run -d --name sonarqube -p 9000:9000 sonarqube:9.2-community
                        echo "Waiting for SonarQube to be ready..."
                        while ! curl -s http://localhost:9000/api/system/health | grep '"status":"UP"'; do sleep 5; done
                    '''
                    sh './gradlew sonarqube -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN'
                    sh 'docker stop sonarqube'
                    sh 'docker rm sonarqube'
                }
            }
        }
    }
}

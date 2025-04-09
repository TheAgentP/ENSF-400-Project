pipeline{
    agent any

    // Set environment variables for the image
    environment {
        IMAGE_NAME = 'ensf400-project'
        TAG = 'latest'
    }

    stages{
        // Building the image itself
        stage('Build'){
            steps{
                sh 'docker build -t $IMAGE_NAME:$TAG .'
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

        //Use a docker in docker container to run sonarcube (can't figure out)
        stage('Static Analysis') {
            agent {
                docker {
                    image 'docker:20.10.7-dind'  // Docker-in-Docker container
                    args '--privileged'  // Enable Docker to run inside the container
                }
            }
            steps {
                script {
                    // Cleanup any existing SonarQube containers
                    sh '''
                        docker ps -a -q --filter "name=sonarqube" | xargs -r docker stop | xargs -r docker rm
                    '''

                    // Start SonarQube in a Docker-in-Docker container
                    sh '''
                        docker run -d --name sonarqube -p 9000:9000 sonarqube:9.2-community
                        echo "Waiting for SonarQube to start..."
                        sleep 30  # Give SonarQube time to start up
                    '''

                    // Run the static analysis with Gradle
                    sh './gradlew sonarqube'  // Use localhost to access SonarQube
                    sleep 5  // Optional: wait for SonarQube to finish analysis
                    sh './gradlew checkQualityGate'  // Ensure quality gate is passed
                    sh 'docker stop sonarcube'
                }
            }
        }
    }
}

// test
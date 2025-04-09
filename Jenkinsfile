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
                    image 'docker:20.10.7-dind'
                    args '--privileged'
                }
            }
            steps {
                script {
                    // Clean up existing SonarQube containers
                    sh '''
                        docker ps -a -q --filter "name=sonarqube" | xargs -r docker rm -f
                    '''

                    // Start SonarQube
                    sh '''
                        docker run -d --name sonarqube -p 9000:9000 sonarqube:9.2-community
                        echo "Waiting for SonarQube to fully start..."
                        sleep 60
                    '''

                    // Run analysis
                    sh './gradlew sonarqube -Dsonar.host.url=http://localhost:9000 -Dsonar.login=admin -Dsonar.password=admin'
                }
            }
        }

    }
}

// test4
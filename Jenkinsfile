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
                    image 'gradle:7.6.1-jdk11' // ✅ This image includes both Gradle & Java
                }
            }
            steps {
                script {
                    // Run SonarQube in the background (outside the container)
                    sh '''
                        docker rm -f sonarqube || true
                        docker run -d --name sonarqube -p 9000:9000 sonarqube:9.2-community
                        echo "Waiting for SonarQube to start..."
                        sleep 60
                    '''

                    // Run static analysis using Gradle and Java inside the container
                    sh './gradlew sonarqube -Dsonar.host.url=http://localhost:9000 -Dsonar.login=admin -Dsonar.password=admin'
                }
            }
        }
    }
}

// test4
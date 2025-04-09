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
            steps {
                script {
                    // Start SonarQube server
                    sh 'docker run -d --name sonarqube -e SONAR_FORCEAUTHENTICATION=false -p 9000:9000 --pull always sonarqube'
                    // sh 'docker run -d --name sonarqube -p 9000:9000 -e SONAR_FORCEAUTHENTICATION=false sonarqube:9.2-community'

                    sh 'echo "Waiting for SonarQube to start..." && sleep 30'

                    // Build the SonarQube analysis image
                    sh 'docker build -t sonarqube-analysis -f Dockerfile.sonarqube-analysis .'

                    // Run the analysis
                    sh 'docker run --network="host" sonarqube-analysis'

                    // Optional: Check quality gate
                    // sh 'docker run --network="host" sonarqube-analysis ./gradlew checkQualityGate || true'
                }
            }
        }
    }
}

// test4
pipeline{
    agent any 
    stages{
        stage("Sonar Quality Check"){
            agent{
                docker {
                    image 'openjdk'
                }
            }
            steps{
                script{ 
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube --info'
                    }
                }
            }
        }
    }
}
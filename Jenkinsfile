pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{ 
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube --info'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage("Docker Build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_docker_repo_pass', variable: 'nexus_docker_repo_pass_var')]) {
                            sh '''
                            docker build -t 52.66.172.21:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_docker_repo_pass_var 52.66.172.21:8083
                            docker push 52.66.172.21:8083/springapp:${VERSION}
                            docker rmi 52.66.172.21:8083/springapp:${VERSION}
                            '''
                }
                    
                }
            }
        }
    }
}
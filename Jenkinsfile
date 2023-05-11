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
                    timeout(time: 15, unit: 'MINUTES') {
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
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                            sh '''
                            docker build -t 15.206.89.243:8083/myapp:${VERSION} .
                            docker login -u admin -p $nexus_pass_var 15.206.89.243:8083
                            docker push 15.206.89.243:8083/myapp:${VERSION}
                            docker rmi 15.206.89.243:8083/myapp:${VERSION}
                            '''
                }
                }
            }
        }
        
    }
    post {
	    always {
	        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";
		}
    post {
        always {
            slackSend channel: 'jenkins-bot', teamDomain: 'devops-prep', tokenCredentialId: 'slack-token'
            slackSend "Project: ${env.JOB_NAME} Build deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
}
}

}
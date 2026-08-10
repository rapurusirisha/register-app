pipeline {
    agent { label 'Jenkins-agent' }
    
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/rapurusirisha/register-app.git'
                }
        }    
        stage("build Application"){
            steps{
                sh "mvn clean package"
            }
        }
   }
}

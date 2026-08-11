pipeline {
    agent { label 'Jenkins-agent' }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/rapurusirisha/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("Test Application") {
            steps {
                sh 'mvn test'
            }
        }

        stage("SonarQube Code Quality Check") {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=register-app \
                        -Dsonar.projectName=register-app
                        '''
                    }
                }
            }
        }
    }
}

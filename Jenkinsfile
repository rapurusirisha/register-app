pipeline {
    agent { label 'Jenkins-agent' }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"

        DOCKER_USER = "rapurusirisha"
        DOCKER_PASS = 'Dockerhub'

        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

        // Nexus Details
        NEXUS_URL = "http://52.66.28.176:8081"
        NEXUS_SNAPSHOT_REPO = "maven-snapshots"
        NEXUS_RELEASE_REPO = "maven-releases"
    }

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

        stage("Deploy to Nexus") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus',
                        usernameVariable: 'admin',
                        passwordVariable: 'admin123'
                    )
                ]) {

                    sh '''
                        cat > settings.xml <<EOF
<settings>
    <servers>

        <server>
            <id>nexus-snapshots</id>
            <username>${NEXUS_USERNAME}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>

        <server>
            <id>nexus-releases</id>
            <username>${NEXUS_USERNAME}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>

    </servers>
</settings>
EOF

                        mvn deploy \
                          -s settings.xml \
                          -DaltDeploymentRepository=nexus-snapshots::${NEXUS_URL}/repository/${NEXUS_SNAPSHOT_REPO}/

                        rm -f settings.xml
                    '''
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
    }
}

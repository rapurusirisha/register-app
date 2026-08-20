pipeline {
    agent { label 'Jenkins-agent' }

    environment {
        // Application
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"

        // Docker Hub
        DOCKER_USER = "rapurusirisha"
        DOCKER_PASS = "Dockerhub"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

        // Nexus
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
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
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

        stage("Build Docker Image") {
            steps {
                script {
                    docker_image = docker.build "${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }

        stage("Deploy Application") {
            steps {
                sh """
                    echo "Pulling Docker image..."
                    docker pull ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Stopping old container..."
                    docker stop ${APP_NAME} || true

                    echo "Removing old container..."
                    docker rm ${APP_NAME} || true

                    echo "Starting new container..."
                    docker run -d \
                        --name ${APP_NAME} \
                        -p 8081:8080 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Container status:"
                    docker ps
                """
            }
        }
    }

    post {
        success {
            echo "========================================="
            echo "Pipeline completed successfully!"
            echo "Application: ${APP_NAME}"
            echo "Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo "Application URL: http://<SERVER-IP>:8081"
            echo "========================================="
        }

        failure {
            echo "Pipeline failed. Please check the stage logs."
        }

        always {
            sh 'rm -f settings.xml || true'
        }
    }
    stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user SirishaRapuru:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-206-196-173.ap-south-1.compute.amazonaws.com:8080/job/CDpipeline/buildWithParameters?token=newcluster'"
                }
            }
       }
}

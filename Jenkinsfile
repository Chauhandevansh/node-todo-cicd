pipeline {
    agent { label 'web-agent' }
    environment {
        REGISTRY       = "devanshchauhan04"
        FRONTEND_IMAGE = "${REGISTRY}/mern-frontend"
        BACKEND_IMAGE  = "${REGISTRY}/mern-backend"
        COMPOSE_FILE   = "docker-compose.yml"
    }
    stages {
        stage("Clone Repository") {
            steps {
                git branch: "main", url: "https://github.com/Chauhandevansh/MERN-docker-compose.git"
            }
        }

        stage("Build Frontend Image") {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage("Build Backend Image") {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage("Push Images to Registry") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHubCreds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                        docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage("Deploy") {
            steps {
                sh """
                    FRONTEND_IMAGE=${FRONTEND_IMAGE}:${BUILD_NUMBER} \
                    BACKEND_IMAGE=${BACKEND_IMAGE}:${BUILD_NUMBER} \
                    docker compose down && docker compose up -d --build
                """
            }
        }

        stage("Cleanup") {
            steps {
                sh """
                    echo "Cleaning up unused Docker resources..."
                    docker image prune -af || true
                    docker container prune -f || true
                    docker volume prune -f || true
                """
            }
        }
    }
}

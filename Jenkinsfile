pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_REPO = 'tumenta3322/student-management'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker version'
            }
        }

        stage('Build image') {
            steps {
                sh '''
                    set -eu
                    docker build \
                        -t "${IMAGE_REPO}:jenkins-${BUILD_NUMBER}" .
                '''
            }
        }

        stage('Test container') {
            steps {
                sh '''
                    set -eu

                    container="student-migration-test-${BUILD_NUMBER}"

                    cleanup() {
                        docker rm -f "$container" >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker run -d --name "$container" \
                        "${IMAGE_REPO}:jenkins-${BUILD_NUMBER}"

                    attempt=0
                    until docker exec "$container" \
                        wget -q -O /tmp/response.html http://127.0.0.1/; do
                        attempt=$((attempt + 1))

                        if [ "$attempt" -ge 15 ]; then
                            docker logs "$container"
                            exit 1
                        fi

                        sleep 2
                    done

                    docker cp "$container:/tmp/response.html" response.html
                    cmp index.html response.html

                    echo "PASS: Nginx served the exact page from this build."
                '''
            }
        }

        stage('Publish image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-student',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x

                        export DOCKER_CONFIG="$(mktemp -d)"

                        cleanup() {
                            rm -rf "$DOCKER_CONFIG"
                        }
                        trap cleanup EXIT

                        printf '%s' "$DOCKERHUB_TOKEN" |
                            docker login \
                                --username "$DOCKERHUB_USER" \
                                --password-stdin

                        docker push "${IMAGE_REPO}:jenkins-${BUILD_NUMBER}"
                    '''
                }
            }
        }
    }
}
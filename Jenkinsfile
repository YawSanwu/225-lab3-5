
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/sanwuya'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/YawSanwu/225-lab3-5.git'
        KUBECONFIG = credentials('sanwuya-225')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]]
                ])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(
                        "${DOCKER_IMAGE}:${IMAGE_TAG}",
                        "-f Dockerfile.build ."
                    )
                }
            }
        }

        stage('Verify Website Files') {
    steps {
        sh '''
            docker run --rm --entrypoint sh \
              ${DOCKER_IMAGE}:${IMAGE_TAG} \
              -c 'ls -la /usr/share/nginx/html/lab3-5/ && nginx -T 2>&1 | grep -A 20 "server {"'
        '''
    }
}
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        "${DOCKER_CREDENTIALS_ID}"
                    ) {
                        docker.image(
                            "${DOCKER_IMAGE}:${IMAGE_TAG}"
                        ).push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                sh """
                    sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml
                    kubectl apply -f deployment-dev.yaml
                """
            }
        }

        stage('Deploy to Prod Environment') {
            steps {
                sh """
                    sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-prod.yaml
                    kubectl apply -f deployment-prod.yaml
                """
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                sh 'kubectl get all'
            }
        }
    }

    post {
        success {
            slackSend color: 'good',
                message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }

        unstable {
            slackSend color: 'warning',
                message: "Build Unstable: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }

        failure {
            slackSend color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}

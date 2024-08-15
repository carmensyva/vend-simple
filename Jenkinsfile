pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'vend-simple' // Replace with your Docker image name
        GIT_REPO = 'https://github.com/carmensyva/vend-simple.git'
        BRANCH = 'main'
        CREDENTIALS_DOCKER = 'dockerhub' // Jenkins credential ID
        CREDENTIALS_GIT = 'cred'
        MANIFEST_PATH = 'openshift/config/deployment.yaml' // Path to the manifest file
    }

    stages {
        stage('Clone'} {
            step {
                script {
                    sh "git config --global http.sslVerify false"
                    withCredentials([usernamePassword(credentialsId: 'cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "git clone https://\${USERNAME}:\${PASSWORD}@${GIT_REPO} source "
                    }
                    sh "git fetch"
                    sh "git switch ${BRANCH}"
                    COMMIT_ID = sh(script: 'git rev-parse --short=8 HEAD').trim()
                    env.DOCKER_TAG = "${COMMIT_ID}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the commit ID as the tag
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Use the credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_DOCKER}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // Login to DockerHub or other registry
                        sh 'echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin'

                        // Push the Docker image to the registry
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Update Manifest') {
            steps {
                script {
                    // Update the image URL in the manifest file
                    sh 'sed -i \'s|image: .*\\$|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|\' ${MANIFEST_PATH}'

                    // Commit and push the changes back to the repository
                    withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_GIT}", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git config user.name "carmensyva"
                            git config user.email "jenkins@example.com"
                            git add ${MANIFEST_PATH}
                            git commit -m "Update image URL to ${DOCKER_IMAGE}:${DOCKER_TAG}"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO}
                        """
                    }
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "my-node-app-image"
        GIT_REPO_NAME = 'Deploy-a-Node.js-App-using-Jenkins-and-ArgoCD-on-Ubuntu'
        GIT_MANIFESTS_REPO_NAME = 'ArgoCD-Manifests-For-Node.js-App'
        GIT_USERNAME = 'awsaf-utm'
        GIT_USER_EMAIL = 'engzaman2020@gmail.com'
        DOCKERHUB_USERNAME = 'engzaman2020'
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/${GIT_USERNAME}/${GIT_REPO_NAME}.git",
                branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh 'docker build -t ${DOCKERHUB_USERNAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh """
                        echo 'Push to Repo'
                        docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github-token', 
                url: "https://github.com/${GIT_USERNAME}/${GIT_MANIFESTS_REPO_NAME}.git",
                branch: 'main'
            }
        }

        stage('Update K8S manifest & push to Repo'){
            steps {
                script {
                    // Retrieve the previous build number
                    def previousBuildNumber = currentBuild.previousBuild?.number
                    echo "Previous Build Number: ${previousBuildNumber}"

                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            ls -al
                            cd dev
                            echo "Original deployment.yaml:"
                            cat deployment.yaml
                            sed -i 's/${previousBuildNumber}/${BUILD_NUMBER}/g' deployment.yaml
                            echo "Original deployment.yaml:"
                            cat deployment.yaml
                            git add deployment.yaml
                            git commit -m 'Updated the deployment yaml | Jenkins Pipeline'
                            git remote -v
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USERNAME}/${GIT_MANIFESTS_REPO_NAME}.git HEAD:main
                        """                        
                    }
                }
            }
        }
    }

    post {
        cleanup {
            // Clean up workspace
            cleanWs()
        }
        always {
            script {
                // Remove Docker image
                sh "docker rmi -f ${DOCKERHUB_USERNAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
            }

            // Notify if necessary, etc.
            echo 'Pipeline finished.'
        }
    }
}

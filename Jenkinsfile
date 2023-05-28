pipeline {

    agent any
    
    environment{

        DOCKERHUB_USERNAME = "kumail7"
        APP_NAME = "gitops-argo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = "dockerhub"
    }


    stages {
        stage ('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage ("Checkout SCM") {
            steps {
                git credentialsId: 'github',
                url: 'https://github.com/kumailr7/argo-gitops-project',
                branch: 'main'
            }
        }

        stage ("Docker Build Image") {
            steps {
                script {

                    docker_image = docker.build "${IMAGE_NAME}"

                }
            }
        }

        stage ("Docker Push the image") {
            steps {
                script {

                    docker.withRegistry('',REGISTRY_CREDS){
                        docker_image.push("$BUILD_NUMBER")
                        docker_image.push("latest")
                    }

                }
            }
        }

        stage (" Deleting the Docker images") {
            steps {
                script {
                
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"

                }
            }
        }

        stage ("Updating the Kubernetes deployment file") {
            steps {
                script {

                    sh """
                     cat deployment.yml
                     sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml
                     cat deployment.yml  
                    """

                }
            }
        }

        stage ("Pushing the changes in deployment file to GitHub Repo") {
            steps {
                script {
                   
                   sh """
                    git config --global user.name "kumailr7"
                    git config --global user.email "kumailrizvi70@gmail.com"
                    git add deployment.yml
                    git commit -m "updated the deployment file"     
                   """
                   withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        
                        sh "git push https://github.com/kumailr7/argo-gitops-project.git main"

                    }

                }
            }
        }
    }

}
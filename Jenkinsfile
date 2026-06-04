pipeline {
    agent any 
        options {
            disableConcurrentBuilds()
        }
        environment {
            DOCKER_IMAGE = "jinendra7545/myapp"
            GIT_USERNAME = "jinendra7545"
            GIT_EMAIL = "devopsprob10@gmail.com"
        }
        stages {
            stage('GitCheckOut'){
                steps {
                    checkout scm
                }
            }
            stage('Build and Push Image'){
                when{ branch 'main' }
                steps{
                    script{
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]){
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        """
                    }
                    
                }
                }
            }
            stage('Update the Manifest'){
                when { branch 'main' }
                steps {
                    script{
                        withCredentials([usernamePassword(
                            credentialsId: 'github-cred',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_TOKEN'
                        )]){
                            sh """
                                set -e 
                                git config --global user.name "${GIT_USERNAME}"
                                git config --global user.email "${GIT_EMAIL}"
                                git fetch origin 
                                git checkout main
                                git reset --hard origin/main
                                sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g" k8s/deployment.yml
                                git add k8s/deployment.yml
                                git diff --cached --quiet || git commit -m "Update the image tag to ${IMAGE_TAG}"
                                git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/devops-prob/Multi-Branch-Prod.git main
                            """
                        }
                    }

                }
               
            }
        }
    }

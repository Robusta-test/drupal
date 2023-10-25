pipeline {
    agent any // runs on any available jenkins agent..
    environment {
        DOCKER_IMAGE_NAME = " "
        NAMESPACE = " "
    }
    stages {
        stage('init') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        DOCKER_IMAGE_NAME = "artifactory.vodafone.com:443/docker-vfie-tozi-local/profile-ms-dev"
                        NAMESPACE = ""
                    }
                    else if (env.BRANCH_NAME == "test")
                    {
                        DOCKER_IMAGE_NAME = "artifactory.vodafone.com:443/docker-vfie-tozi-local/profile-ms-test"
                        NAMESPACE = "test"
                    }
                }
            }
        }                    
        stage("build image") {
            steps {
                script {                   
                   withCredentials([usernamePassword(credentialsId: 'Docker', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                      #sudo chmod 777 /var/run/docker.sock
                      docker login -u $USERNAME -p $PASSWORD
                      docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .
                      docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                      """
                   }
                }
            }
        }
   
        stage('Push updates to vf-ie-apps-for-good-tozi ') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

                            }
                }
            }   
          }

        }
    }

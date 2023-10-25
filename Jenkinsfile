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
                    if (env.BRANCH_NAME == "develop") {
                        DOCKER_IMAGE_NAME = "artifactory.vodafone.com:443/docker-vfie-tozi-local/profile-ms-dev"
                        NAMESPACE = "dev"
                    }
                    else if (env.BRANCH_NAME == "test")
                    {
                        DOCKER_IMAGE_NAME = "artifactory.vodafone.com:443/docker-vfie-tozi-local/profile-ms-test"
                        NAMESPACE = "test"
                    }
                }
            }
        }        
        stage("build jar") {
            steps {
                script {  // to be able to write groovy script
                    // echo "building jar version ${MY_VERSION} by ${params.set_name}"
                    echo "building jar "
                    configFileProvider([configFile(fileId: "aa9d0957-27de-42c6-9ad1-f0d63b71e8ae", variable: 'MAVEN_SETTINGS')]) {
                        sh """
                            /usr/local/apache-maven/apache-maven-3.8.7/bin/mvn -s $MAVEN_SETTINGS clean package -DskipTests
                           """
                    }
                }
            }
        }
        stage("SonarQube analysis") {
            steps {
                script {
                   withSonarQubeEnv("SonarQube") { 
                   configFileProvider([configFile(fileId: "aa9d0957-27de-42c6-9ad1-f0d63b71e8ae", variable: 'MAVEN_SETTINGS')]) {
                    sh """
                       /usr/local/apache-maven/apache-maven-3.8.7/bin/mvn -s $MAVEN_SETTINGS clean install -DskipTests sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.projectKey=profile-ms
                      """
                     }
                   }
                }  
            }
        }            
        stage("build image") {
            steps {
                script {                   
                   withCredentials([usernamePassword(credentialsId: 'jfrog_ID', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                      #sudo chmod 777 /var/run/docker.sock
                      docker login artifactory.vodafone.com:443 -u $USERNAME -p $PASSWORD
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
                                sh """
                                    rm -rf vf-ie-apps-for-good-tozi
                                    git clone "https://$USERNAME:$PASSWORD@github.vodafone.com/vf-cdaas/vf-ie-apps-for-good-tozi.git"
                                    cd vf-ie-apps-for-good-tozi/test/${NAMESPACE}/profile-ms
                                    pwd
                                    sed -i '/docker/c\\        image: registry.eu-central-1.harbor.vodafone.com/vf-ie-apps-for-good-tozi/docker-vfie-tozi-local/profile-ms-${NAMESPACE}:${BUILD_NUMBER}' profile-deployment.yaml
                                    #cat /var/lib/jenkins/workspace/profile-ms_${NAMESPACE}/vf-ie-apps-for-good-tozi/test/${NAMESPACE}/profile-ms/profile-deployment.yaml
                                    ls -a 
                                    cd /var/lib/jenkins/workspace/profile-ms_${NAMESPACE}/vf-ie-apps-for-good-tozi/
                                    #echo "  - artifactory.vodafone.com/docker-vfie-tozi-local/profile-ms-${NAMESPACE}:${BUILD_NUMBER}" >> docker_images.yml
                                    sed -i '/^docker_images:.*/a \\ \\ - artifactory.vodafone.com/docker-vfie-tozi-local/profile-ms-${NAMESPACE}:${BUILD_NUMBER}' docker_images.yml
                                    git config --global user.email "ahmed.gamalmansour1@vodafone.com"
                                    git config --global user.name "ahmed-gamalmansour1"
                                    git config pull.rebase true 
                                    git add .
                                    git commit -m "profile-deploymnet.yaml image updated"
                                    git push origin main
                                """
                            }
                }
            }   
          }

        }
    }

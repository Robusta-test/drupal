pipeline {
  tools {
    // a bit ugly because there is no `@Symbol` annotation for the DockerTool
    // see the discussion about this in PR 77 and PR 52: 
    // https://github.com/jenkinsci/docker-commons-plugin/pull/77#discussion_r280910822
    // https://github.com/jenkinsci/docker-commons-plugin/pull/52
    dockerTool 'docker-19.03.11'
  }    
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
                        DOCKER_IMAGE_NAME = "ahmedgmansour/drupal"
                        NAMESPACE = ""
                    }
                    else if (env.BRANCH_NAME == "test")
                    {
                        DOCKER_IMAGE_NAME = "ahmedgmansour/drupal-test"
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
                      echo $PATH
                      docker login -u $USERNAME -p $PASSWORD
                      docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .
                      docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                      """
                  // docker.withRegistry('https://docker.io', 'Docker') {
                  //   def image = docker.build('ahmedgmansour/drupal:39 .') 
                  //   image.push()
                     }                    
                   }
                }
            } 

        }
    environment {
    PATH = '/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/var/jenkins_home/tools/org.jenkinsci.plugins.docker.commons.tools.DockerTool/docker-19.03.11/bin'

      }
    }

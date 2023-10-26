pipeline { 
  // tools {
  //   // a bit ugly because there is no `@Symbol` annotation for the DockerTool
  //   // see the discussion about this in PR 77 and PR 52: 
  //   // https://github.com/jenkinsci/docker-commons-plugin/pull/77#discussion_r280910822
  //   // https://github.com/jenkinsci/docker-commons-plugin/pull/52
  //   dockerTool 'docker-19.03.11'
  // }    
    agent 
      kubernetes {
      yamlFile 'builder.yaml'
    }
    environment {
        DOCKER_IMAGE_NAME = " "
        NAMESPACE = " "

    }
    stages {
        stage('init') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        DOCKER_IMAGE_NAME = "docker.io/ahmedgmansour/drupal"
                        NAMESPACE = ""
                    }
                    else if (env.BRANCH_NAME == "test")
                    {
                        DOCKER_IMAGE_NAME = "docker.io/ahmedgmansour/drupal-test"
                        NAMESPACE = "test"
                    }
                }
            }
        }                    
    stage('Kaniko Build & Push Image') {
      steps {
        container('kaniko') {
          script {
            sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                             --context `pwd` \
                             --destination=ahmedgmansour/myweb:${BUILD_NUMBER}
            '''
            }
            }
          }
         } 
        }
    }

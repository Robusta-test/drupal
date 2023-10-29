pipeline { 
  // tools {
  //   // a bit ugly because there is no `@Symbol` annotation for the DockerTool
  //   // see the discussion about this in PR 77 and PR 52: 
  //   // https://github.com/jenkinsci/docker-commons-plugin/pull/77#discussion_r280910822
  //   // https://github.com/jenkinsci/docker-commons-plugin/pull/52
  //   dockerTool 'docker-19.03.11'
  // }    
    agent {
      kubernetes {
        yamlFile 'builder.yaml'
      }
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
    stage('Kaniko Build & Push Image') {
      steps {
        container('kaniko') {
          script {
            sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                             --context `pwd` \
                             --destination=ahmedgmansour/drupal:${BUILD_NUMBER}
            '''
            }
            }
          }
         }
    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
          script {
            kubernetesDeploy(
              configs: 'drupal/drupal/values.yaml',  
              kubeconfigId: 'mykubeconfig'
            )
          // withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
          //   sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
          //   sh 'kubectl apply -f myweb.yaml'
          }
        }
      }
    }      
    stage('Push updates to Robusta-helm ') {
      steps {
         script {
             withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                sh """
                                    rm -rf Robusta-helm
                                    git clone "https://$USERNAME:$PASSWORD@github.com/ahmedgmansour/Robusta-helm.git"
                                    cd Robusta-helm
                                    sed -i '/drupal/c\\        image: ahmedgmansour/drupal:${BUILD_NUMBER}'  drupal/drupal/vaules.yaml
                                    ls -a
                                    git config --global user.email "ahmedgmal.383#gmail.com"
                                    git config --global user.name "ahmedgmansour"
                                    git config pull.rebase true 
                                    git add .
                                    git commit -m "vaules.yaml image updated"
                                    git push origin main
                                """
                  }
                }
            }   
          }       
        }
    }

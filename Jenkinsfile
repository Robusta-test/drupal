pipeline{
    agent any
    // tools{
    //     jdk 'jdk17'
    //     nodejs 'node16'
    // }
    environment {
        DOCKER_IMAGE_NAME = " "
        // NAMESPACE = " "

    }
    stages {
        stage('init') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        DOCKER_IMAGE_NAME = "ahmedgmansour/drupal"
                        // NAMESPACE = ""
                    }
                    else if (env.BRANCH_NAME == "diamond")
                    {
                        DOCKER_IMAGE_NAME = "ahmedgmansour/drupal-test"
                        // NAMESPACE = "test"
                    }
                }
            }
        }                    
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'diamond', url: 'https://github.com/Robusta-test/drupal.git'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t $DOCKER_IMAGE_NAME ."
                       sh "docker tag $DOCKER_IMAGE_NAME:latest "
                       sh "docker push $DOCKER_IMAGE_NAME:latest "
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name drupal -p 9080:80 ahmedgmansour/$DOCKER_IMAGE_NAME:latest'
            }
        }
    }
  }
  

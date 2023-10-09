pipeline {
    agent {
        docker {
          image 'chaitannyaa/maven-plus-docker'
          args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
    stages {
        stage('Build and Test') {
            steps {
                script {
                    // Build the project and create a JAR file
                    sh 'mvn clean package'
                }
            }
        }
        stage('Code Analysis with SonarQube') {
            environment {
                SONAR_URL = 'http://100.25.200.83:9000'
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        sh "mvn sonar:sonar -Dsonar.login=\$SONAR_AUTH_TOKEN -Dsonar.host.url=\$SONAR_URL"
                    }
                }
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "samuelsamits/java_awesome-cicd:${BUILD_NUMBER}"
            }
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t \$DOCKER_IMAGE ."
                    
                    // Push Docker image to a registry (assuming Docker Hub)
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u \$DOCKERHUB_USERNAME -p \$DOCKERHUB_PASSWORD"
                        sh "docker push \$DOCKER_IMAGE"
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "Jenkins_ArgoCD_Sonarcube_Java_Webapp_K8s"
                GIT_USER_NAME = "samuelsamits"
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "samuelsamits@gmail.com"
                            git config user.name "samuelsamits"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            sed -i "s/replaceImageTag/\$BUILD_NUMBER/g" manifests/deployment.yml
                            git add manifests/deployment.yml
                            git add target/
                            git commit -m "Update image version \$BUILD_NUMBER"
                            git push https://$GITHUB_TOKEN@github.com/$GIT_USER_NAME/$GIT_REPO_NAME HEAD:main
                        '''
                    }
                }
            }
        }
    }
}

pipeline {
    agent {
        docker {
            image 'chaitannyaa/maven-plus-docker'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
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
        stage('Update Deployment File and Commit Changes') {
            environment {
                GIT_REPO_NAME = "Jenkins_ArgoCD_Sonarcube_Java_Webapp_K8s"
                GIT_USER_NAME = "samuelsamits"
            }
            steps {
                script {
                    // Set Git user information
                    sh "git config user.email 'samuelsamits@gmail.com'"
                    sh "git config user.name 'samuelsamits'"
                    
                    // Update the image tag in the deployment file
                    def buildNumber = BUILD_NUMBER
                    sh "sed -i 's/replaceImageTag/\$BUILD_NUMBER/g' manifests/deployment.yml"
                    
                    // Check if there are changes to commit
                    def hasChanges = sh(returnStatus: true, script: "git diff --quiet --exit-code manifests/deployment.yml")
                    
                    if (hasChanges == 0) {
                        // Add the modified file to the Git staging area
                        try {
                            sh "git add manifests/deployment.yml"
                        } catch (Exception e) {
                            error "Failed to add the file to the Git staging area: ${e.message}"
                        }
                        
                        // Commit changes
                        try {
                            sh "git commit -m 'Update image version \$BUILD_NUMBER'"
                        } catch (Exception e) {
                            error "Failed to commit changes: ${e.message}"
                        }
                        
                        // Push changes to the remote repository
                        try {
                            sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main"
                        } catch (Exception e) {
                            error "Failed to push changes to the remote repository: ${e.message}"
                        }
                    } else {
                        echo "No changes to commit."
                    }
                }
            }
        }
    }
}

pipeline {
    agent any
    environment {
        SCANNER_HOME= tool 'sonar-scaner'
        DOCKER_IMAGE = "mu221/first-cicd:${BUILD_NUMBER}"
        
        
    }
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('check out from SCM') {
            steps {
                git branch: 'code', url: 'https://github.com/Muralidhar-br/cicd.git'
                
            }
        }
        stage('compile application') {
            steps {
                sh 'mvn compile'
                
            }
        }
        stage('test application') {
            steps {
                sh 'mvn test'
                
            }
        }
        stage('file system scan') {
            steps {
                sh "trivy fs -f table -o fs.html ."
                
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh ''' ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=first -Dsonar.projectKey=first \
                -Dsonar.java.binaries=. '''
}
            }
        }
          stage("Quality Gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonartoken'
                }
            }
        }
         stage("build application"){
            steps {
                sh 'mvn package'
            }
        }
        stage("build and tag docker image"){
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockertoken', toolName: 'docker') {
                        sh 'docker build -t ${DOCKER_IMAGE} .'
                    }
                }
                
            }
        }
        stage('docker image scan') {
            steps {
                sh "trivy image -f table -o image.html ${DOCKER_IMAGE}"
                
            }
        }
        stage('push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockertoken', toolName: 'docker') {
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                
                }
            }
        }
        stage('Update Deployment File') {
            environment {
            GIT_REPO_NAME = "cicd"
            GIT_USER_NAME = "Muralidhar-br"
            APP_NAME = "first-cicd"
            }
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'git-token', gitToolName: 'git')]) {
                    sh '''
                        git config user.email "mbr22297@gmail.com"
                        git config user.name "muralidhar"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s/${APP_NAME}:.*/${APP_NAME}:${BUILD_NUMBER}/g" dep/deployment.yaml
                        git add dep/deployment.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${git-token}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:code
                    '''
                }
            }
        }
        
    }
}


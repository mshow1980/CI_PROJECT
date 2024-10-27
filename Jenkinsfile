pipeline {
    agent any
    tools {
        nodejs 'npm23'
        jdk     'jdk17' 
    }
    environment {
        SCANNER_HOME = tool 'SonarQube_Scanner'
        DOCKER_USER = 'mshow1980'
        APP_NAME  = 'reditt-app'
        IMAGE_NAME = "${DOCKER_USER}"+"/"+"${ APP_NAME}"
        VERSION = '0.3'
        IMAGE_TAG = "${VERSION}-${BUILD_NUMBER}"
        REGISTRY_CRED = 'Docker-token'
    }
    stages{
        stage('CLEANWS'){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Checkout-SCM'){
            steps{
                script{
                   checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mshow1980/CI_PROJECT.git']]) 
                }
            }
        }
        stage('Install Dependencies'){
            steps{
                script{
                    sh 'npm install'
                }
            }
        }
        stage('SOnarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv('SonarQube_server') {
                        sh """
                        $SCANNER_HOME/bin/SonarQube_Scanner -Dsonar.projectName=reditt-app\
                        -Dsonar.projectKey=reditt-app
                        """
                    }
                }
            }
        }
        stage('Quality_Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'SOnar-Token'
                }
            }
        }
    }
}

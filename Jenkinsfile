pipeline {
    agent any
    tools {
        nodejs 'npm23'
        jdk     'jdk17' 
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_USER = 'mshow1980'
        APP_NAME  = 'reditt-app'
        IMAGE_NAME = "${DOCKER_USER}"+"/"+"${ APP_NAME}"
        VERSION = '0.3'
        IMAGE_TAG = "${VERSION}-${BUILD_NUMBER}"
        REGISTRY_CRED = 'Docker_token'
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
        stage('OWASP Dependency Check'){
            steps{
                script{
                    dependencyCheck additionalArguments: 
                    ''' -o "./"
                        -s "./"
                        -f "ALL"
                        --prettyPrint''', odcInstallation: 'OWASP_DC'
                    dependencyCheckPublisher pattern: 'dependencyCheck-report.xml'
                }
            }
        }
        stage('TRIVY FIle Scan'){
            steps{
                script{
                    sh "trivy fs --format table -o mickey.html ."
                }
            }
        }
        stage('SOnarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv('SonarQube_servers') {
                        sh """
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-app\
                        -Dsonar.projectKey=reddit-app
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
        stage('Install Dependencies'){
            steps{
                script{ 
                    sh "npm install"  
                }
            }
        }
        stage('Docker build'){
            steps{
                script{ 
                    withDockerRegistry(credentialsId: 'Docker_token', toolName: 'doker') {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Trivy Image Scan'){
            steps{
                script{ 
                    sh "trivy image ${docker_image} --format table -o tryvyimage.html" 
                }
            }
        }
    }
}

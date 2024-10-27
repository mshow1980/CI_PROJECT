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
        REGISTRY_CRED = 'docker-login'
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
                    withDockerRegistry(credentialsId: 'docker-login', toolName: 'doker') {
                        docker_image = docker.build"${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Trivy Image Scan'){
            steps{
                script{ 

                    sh 'trivy image --scanners vuln --scanners misconfig "${IMAGE_NAME}" --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table -o trvy-image.html ' 
                }
            }
        }
        stage('Image Tag& Push'){
            steps{
                script{ 
                    withDockerRegistry(credentialsId: 'docker-login') {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')

                    }  
                }
            }
        }
        stage('Updating Image Tag'){
            steps{
                script{ 
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                         sh ''' 
                            cat manifest.yaml
                            sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' manifest.yaml
                            cat manifest.yaml
                        '''
                    }
                }
            }
        }
        stage('Upadting Manifest'){
            steps{
                script{ 
                    sh '''
                    git config --global user.name "SCION_SCOPE"
                    git config --global user.email "mshow1980@aol.com"
                    git add manifest.yaml
                    git commit -m 'updated manifest'
                    git push origin main
                    echo "done!!"
                    '''
                    sh 'git push https://github.com/mshow1980/CD_PROJECT_ARGOCD.git main'
                }
            }
        } 
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'scionventureslls@gmail.com',                              
            attachmentsPattern: 'mickey.html,trvy-image.html'
        }
    }
}

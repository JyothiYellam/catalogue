pipeline {
    agent {
        node {
            label "ROBOSHOP"
        }
    }  
    environment {
        appVersion  = ""
        ACC_ID = "423389601241"
        region = "us-east-1"
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 5, unit: 'MINUTES')
    }
    /* parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        booleanParam(name: 'DEPLOY', defaultValue: true, description: 'Toggle this value')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    } */
    stages {
        stage('Read version'){
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    // Access fields direclty

                    appVersion = packageJson.version     
                    echo "Building version ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script{
                    sh """ 
                        npm install
                    """
                }
            }
        }
        stage('Build Image') {
            steps {
                script{
                     withAWS(credentials: 'aws-creds', region: "${region}") {
                      // commands here have  AWS authentication
                        sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} 
                        """
                    }
                }
            }    
        }
    }
    
    // post build
    post {
        always {
            echo 'I will always say Hello again!' 
        }
        success {
            echo "pipeline success"
        }
        failure {
            echo "pipeline failure" 
        }
    } 
}
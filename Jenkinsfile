pipeline {
    agent {
        node {
            label "roboshop"
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
        stage('Unit tests'){
            steps {
                script {
                    sh """
                        npm test
                    """
                }
            }
        }
        /* stage('SonarQube Analysis'){
            steps {
               script {
                    def scannerHome = tool name: 'sonar-8'   // agent configuration
                    withSonarQubeEnv('sonar-server') {  // analysing and uplodaing to server
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }

        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        } */
        stage('Dependabot Alerts Check') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def owner = 'JyothiYellam'
                        def repo  = 'catalogue'

                        // Fetch high + critical Dependabot alerts via GitHub REST API
                        def response = sh(
                            script: """
                                curl -s -w "\\n%{http_code}" \\
                                -H "Authorization: Bearer ${GITHUB_TOKEN}" \\
                                -H "Accept: application/vnd.github+json" \\
                                -H "X-GitHub-Api-Version: 2022-11-28" \\
                                "https://api.github.com/repos/${owner}/${repo}/dependabot/alerts?state=open&per_page=100"
                            """,
                            returnStdout: true
                        ).trim()

                        // Split body and HTTP status code
                        def parts      = response.tokenize('\n')
                        def httpStatus = parts[-1].trim()
                        def body       = parts[0..-2].join('\n')

                        if (httpStatus != '200') {
                            error "GitHub API call failed with HTTP ${httpStatus}. Check GITHUB_TOKEN permissions."
                        }

                        // Parse the JSON response
                        def alerts = readJSON text: body

                        // Filter for high + critical severity, open alerts only
                        def criticalHigh = alerts.findAll {
                            it.security_advisory.severity == 'high' || it.security_advisory.severity == 'critical'
                        }

                        echo "Found ${criticalHigh.size()} open High/Critical Dependabot alert(s)"

                        if (criticalHigh.size() > 0) {
                            echo "--------------------------------------------------"
                            echo "HIGH/CRITICAL VULNERABILITIES DETECTED:"
                            criticalHigh.each {
                                echo "- [${it.security_advisory.severity.toUpperCase()}] ${it.security_advisory.summary} (package: ${it.dependency.package.name})"
                            }
                            echo "--------------------------------------------------"
                            error "Failing pipeline due to unresolved High/Critical Dependabot alerts."
                        }

                    echo "No High/Critical Dependabot alerts found. Proceeding."
                    }
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
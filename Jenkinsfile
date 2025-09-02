pipeline {
    agent any
    
    tools {
        python 'Python3'
    }
    
    environment {
        PROJECT_KEY = 'OT-MICROSERVICES-attendance-api'
        PROJECT_NAME = 'OT-MICROSERVICES Attendance API'
        SONAR_URL = 'http://localhost:9000'
        REPO_URL = 'https://github.com/OT-MICROSERVICES/attendance-api.git'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${tool 'SonarScanner'}/bin/sonar-scanner \
                        -Dsonar.projectKey=${PROJECT_KEY} \
                        -Dsonar.projectName='${PROJECT_NAME}' \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.python.version=3 \
                        -Dsonar.exclusions=**/venv/**,**/migrations/**,**/*.pyc,**/__pycache__/**,**/tests/**,setup.py \
                        -Dsonar.links.homepage=${REPO_URL}
                    """
                }
                
                script {
                    env.SONAR_REPORT_URL = "${SONAR_URL}/dashboard?id=${PROJECT_KEY}"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate abortPipeline: false
                        env.QUALITY_GATE_STATUS = qg.status
                        if (qg.status != 'OK') {
                            echo "Quality gate failed with status: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            emailext (
                subject: "${currentBuild.result ?: 'UNKNOWN'}: SonarQube Analysis for ${PROJECT_NAME}",
                body: """
                <html>
                <body>
                    <h1>SonarQube Analysis Results</h1>
                    <p><b>Project:</b> ${PROJECT_NAME}</p>
                    <p><b>Repository:</b> OT-MICROSERVICES/attendance-api</p>
                    <p><b>Build Status:</b> ${currentBuild.result ?: 'UNKNOWN'}</p>
                    <p><b>Quality Gate Status:</b> ${env.QUALITY_GATE_STATUS ?: 'Not Available'}</p>
                    <p><b>Date & Time:</b> ${new Date().format('yyyy-MM-dd HH:mm:ss', TimeZone.getTimeZone('UTC'))}</p>
                    <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    <p><b>SonarQube Report:</b> <a href="${env.SONAR_REPORT_URL}">View Detailed Report</a></p>
                </body>
                </html>
                """,
                to: "ashutosh.devpro@gmail.com",
                mimeType: 'text/html',
                attachLog: true
            )
            
            cleanWs()
        }
        success {
            echo 'SonarQube analysis completed successfully!'
        }
        failure {
            echo 'SonarQube analysis failed!'
        }
    }
}

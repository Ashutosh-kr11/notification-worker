pipeline {
    agent any
    
    environment {
        PROJECT_KEY = 'OT-MICROSERVICES-Notification-Worker'
        PROJECT_NAME = 'OT-MICROSERVICES Notification Worker'
        SONAR_URL = 'http://localhost:9000'
        REPO_URL = 'https://github.com/OT-MICROSERVICES/notification-worker.git'
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
            script {
                // Set safe defaults for any variables that might not be defined
                def projectName = env.PROJECT_NAME ?: 'Notification Worker'
                def qualityGateStatus = env.QUALITY_GATE_STATUS ?: 'Not Available'
                def sonarReportUrl = env.SONAR_REPORT_URL ?: "${SONAR_URL}/dashboard"
                
                emailext (
                    subject: "${currentBuild.result ?: 'UNKNOWN'}: SonarQube Analysis for ${projectName}",
                    body: """
                    <html>
                    <body>
                        <h1>SonarQube Analysis Results</h1>
                        <p><b>Project:</b> ${projectName}</p>
                        <p><b>Repository:</b> OT-MICROSERVICES/notification-worker</p>
                        <p><b>Build Status:</b> ${currentBuild.result ?: 'UNKNOWN'}</p>
                        <p><b>Quality Gate Status:</b> ${qualityGateStatus}</p>
                        <p><b>Date & Time:</b> ${new Date().format('yyyy-MM-dd HH:mm:ss', TimeZone.getTimeZone('UTC'))}</p>
                        <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <p><b>SonarQube Report:</b> <a href="${sonarReportUrl}">View Detailed Report</a></p>
                    </body>
                    </html>
                    """,
                    to: "ashutosh.devpro@gmail.com",
                    mimeType: 'text/html',
                    attachLog: true
                )
            }
            
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

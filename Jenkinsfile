// Define variables at the top level
def PROJECT_KEY = 'OT-MICROSERVICES-Notification-Worker'
def PROJECT_NAME = 'OT-MICROSERVICES Notification Worker'
def SONAR_URL = 'http://localhost:9000'
def REPO_URL = 'https://github.com/OT-MICROSERVICES/notification-worker.git'
def SONAR_REPORT_URL = ''
def QUALITY_GATE_STATUS = 'Not Available'

// Main script execution
node {
    try {
        stage('Checkout') {
            echo "Checking out repository from ${REPO_URL}"
            git url: REPO_URL, branch: 'main'
        }
        
        stage('SonarQube Analysis') {
            withSonarQubeEnv('SonarQube') {
                def scannerTool = tool 'SonarScanner'
                sh """
                    ${scannerTool}/bin/sonar-scanner \\
                    -Dsonar.projectKey=${PROJECT_KEY} \\
                    -Dsonar.projectName='${PROJECT_NAME}' \\
                    -Dsonar.sources=. \\
                    -Dsonar.sourceEncoding=UTF-8 \\
                    -Dsonar.host.url=${SONAR_URL} \\
                    -Dsonar.python.version=3 \\
                    -Dsonar.exclusions=**/venv/**,**/migrations/**,**/*.pyc,**/__pycache__/**,**/tests/**,setup.py \\
                    -Dsonar.links.homepage=${REPO_URL}
                """
            }
            
            SONAR_REPORT_URL = "${SONAR_URL}/dashboard?id=${PROJECT_KEY}"
        }
        
       /** stage('Quality Gate') {
            timeout(time: 5, unit: 'MINUTES') {
                def qg = waitForQualityGate abortPipeline: false
                QUALITY_GATE_STATUS = qg.status
                if (qg.status != 'OK') {
                    echo "Quality gate failed with status: ${qg.status}"
                }
            }
        } **/
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Build failed: ${e.message}"
        throw e
    } finally {
        // Define safe defaults for variables
        def projectName = PROJECT_NAME ?: 'Notification Worker'
        def qualityGateStatus = QUALITY_GATE_STATUS ?: 'Not Available'
        def sonarReportUrl = SONAR_REPORT_URL ?: "${SONAR_URL}/dashboard"
        
        // Send email with results
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
        
        // Add success/failure specific messages
        if (currentBuild.result == 'SUCCESS') {
            echo 'SonarQube analysis completed successfully!'
        } else if (currentBuild.result == 'FAILURE') {
            echo 'SonarQube analysis failed!'
        }
        
        // Clean workspace
        cleanWs()
    }
}

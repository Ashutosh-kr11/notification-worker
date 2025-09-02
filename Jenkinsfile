pipeline {
    agent any
    
    parameters {
        string(name: 'SONAR_PROJECT_KEY', defaultValue: "${env.JOB_NAME.replaceAll('/', '-')}", description: 'SonarQube project key')
        string(name: 'SONAR_PROJECT_NAME', defaultValue: "${env.JOB_NAME}", description: 'SonarQube project name')
        string(name: 'SONAR_URL', defaultValue: 'http://192.168.1.3:9000', description: 'SonarQube server URL')
        booleanParam(name: 'ENABLE_QUALITY_GATE', defaultValue: true, description: 'Wait for SonarQube quality gate')
        string(name: 'EMAIL_RECIPIENTS', defaultValue: 'ashutosh.devpro@gmail.com', description: 'Email recipients for notifications')
    }
    
    environment {
        PYTHON_VERSION = '3.9'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        PYTHON_HOME = tool "Python-${PYTHON_VERSION}"
        PATH = "${PYTHON_HOME}/bin:${env.PATH}"
        REPOSITORY_URL = sh(script: 'git config --get remote.origin.url', returnStdout: true).trim()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Repository URL: ${env.REPOSITORY_URL}"
            }
        }
        
        stage('Setup Environment') {
            steps {
                sh '''
                    python -m pip install --upgrade pip
                    python -m pip install virtualenv
                    python -m virtualenv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pip install flake8 pylint pytest pytest-cov
                '''
            }
        }
        
        stage('Linting') {
            steps {
                sh '''
                    . venv/bin/activate
                    flake8 --max-line-length=120 --exclude=venv/,migrations/
                    pylint --disable=C0111,R0903,C0103 --max-line-length=120 --ignore=migrations --output-format=parseable $(find . -name "*.py" | grep -v "venv\\|migrations" | xargs)
                '''
            }
        }
        
        stage('Run Tests & Coverage') {
            steps {
                sh '''
                    . venv/bin/activate
                    python -m pytest --cov=. --cov-report=xml:coverage.xml
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'test-results/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=${params.SONAR_PROJECT_KEY} \
                        -Dsonar.projectName='${params.SONAR_PROJECT_NAME}' \
                        -Dsonar.sources=. \
                        -Dsonar.python.coverage.reportPaths=coverage.xml \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=${params.SONAR_URL} \
                        -Dsonar.exclusions=venv/**,**/migrations/**,**/*.pyc,**/__pycache__/**,**/tests/**,setup.py \
                        -Dsonar.links.homepage=${env.REPOSITORY_URL}
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            when {
                expression { params.ENABLE_QUALITY_GATE == true }
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            sh 'rm -rf venv/'
            cleanWs()
        }
        
        success {
            echo 'SonarQube Analysis completed successfully!'
            emailext (
                subject: "SUCCESS: SonarQube Analysis for ${params.SONAR_PROJECT_NAME}",
                body: """<h1>SonarQube Analysis Results</h1>
                <p>Build URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                <p>Project: ${params.SONAR_PROJECT_NAME}</p>
                <p>Repository: <a href='${env.REPOSITORY_URL}'>${env.REPOSITORY_URL}</a></p>
                <p>SonarQube Report: <a href='${params.SONAR_URL}/dashboard?id=${params.SONAR_PROJECT_KEY}'>View Detailed Report</a></p>
                <p>Status: SUCCESS</p>
                """,
                to: "${params.EMAIL_RECIPIENTS}",
                mimeType: 'text/html',
                attachLog: true
            )
        }
        
        failure {
            echo 'SonarQube Analysis failed!'
            emailext (
                subject: "FAILED: SonarQube Analysis for ${params.SONAR_PROJECT_NAME}",
                body: """<h1>SonarQube Analysis Failed</h1>
                <p>Build URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                <p>Project: ${params.SONAR_PROJECT_NAME}</p>
                <p>Repository: <a href='${env.REPOSITORY_URL}'>${env.REPOSITORY_URL}</a></p>
                <p>Status: FAILED</p>
                """,
                to: "${params.EMAIL_RECIPIENTS}",
                mimeType: 'text/html',
                attachLog: true
            )
        }
    }
}

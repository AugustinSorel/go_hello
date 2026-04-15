pipeline {
    agent any

    tools {
        go 'go-1.21'
    }

    environment {
        APP_NAME = "hello-world-api"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the Go API...'
                sh 'GOOS=linux GOARCH=amd64 go build -o ${APP_NAME} main.go'
            }
        }

        stage('Test & Coverage') {
            steps {
                echo 'Running tests...'
                sh 'go test ./...'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('SonarQube-Server') { 
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                      -Dsonar.projectKey=hello-world-go-api \
                      -Dsonar.projectName="Hello World Go API" \
                      -Dsonar.sources=. \
                      -Dsonar.exclusions=**/*_test.go \
                      -Dsonar.tests=. \
                      -Dsonar.test.inclusions=**/*_test.go \
                      -Dsonar.go.coverage.reportPaths=coverage.out
                    '''
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'host.docker.internal:8081',
                    groupId: 'com.example.go',
                    version: "1.0.${env.BUILD_ID}",
                    repository: 'go-binaries',
                    credentialsId: 'nexus-credentials',
                    artifacts: [
                        [artifactId: 'hello-world-api',
                        file: 'hello-world-api',
                        type: 'bin']
                    ]
                )
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Your API is live.'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
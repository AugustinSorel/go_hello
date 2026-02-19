pipeline {
    agent any

    // This loads the Go environment you configured in Jenkins
    tools {
        go 'go-1.21' // Make sure this matches the name in Jenkins Global Tool Configuration
    }

    environment {
        // Defines the output name of your compiled binary
        APP_NAME = "hello-world-api"
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls the latest code from your Git repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the Go API...'
                // Compiles the Go code for a Linux environment
                sh 'GOOS=linux GOARCH=amd64 go build -o ${APP_NAME} main.go'
            }
        }

        stage('Test & Coverage') {
            steps {
                echo 'Running tests and generating coverage report...'
                // The -coverprofile flag generates the file SonarQube needs
                sh 'go test -coverprofile=coverage.out ./...' 
            }
        }

        stage('SonarQube Analysis') {
            environment {
                // This name MUST match what you set in Step 4
                SCANNER_HOME = tool 'SonarScanner' 
            }
            steps {
                // This name MUST match what you set in Step 3
                withSonarQubeEnv('SonarQube') { 
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

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Pauses the pipeline until SonarQube finishes analyzing
                    // Will fail the build if the code is lower quality than allowed
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // The deployment step heavily depends on your infrastructure.
                // Below is an example of copying the binary to a remote server via SSH
                // and restarting a systemd service.
                
                /* sh '''
                scp -o StrictHostKeyChecking=no ${APP_NAME} user@your-server-ip:/opt/api/
                ssh -o StrictHostKeyChecking=no user@your-server-ip "sudo systemctl restart hello-api"
                '''
                */
                
                echo 'Deployment stage complete bite. (Uncomment and configure SSH steps above for actual server deployment)'
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
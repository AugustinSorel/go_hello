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

        stage('Test') {
            steps {
                echo 'Running tests...'
                // If you had a main_test.go, this would run it
                sh 'go test ./...' 
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
                
                echo 'Deployment stage complete. (Uncomment and configure SSH steps above for actual server deployment)'
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
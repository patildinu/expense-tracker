pipeline {
    agent any

    environment {
        // Include npm global binaries in PATH.
        PATH = "/var/lib/jenkins/.npm-global/bin:/usr/bin:/usr/local/bin:$PATH"
        // Define the build output directory as specified in angular.json (adjust if necessary).
        BUILD_DIR = 'dist/expense-tracker'
        // Deployment variables (modify as needed)
        EC2_USER = "ec2-user"
        EC2_IP = "13.202.85.195"
        REMOTE_DIR = "/usr/share/nginx/html"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Clone the expense-tracker repository from GitHub.
                git branch: 'main', url: 'https://github.com/patildinu/expense-tracker'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "Node Version:" && node -v
                    echo "NPM Version:" && npm -v

                    echo "Installing Angular CLI globally..."
                    npm install -g @angular/cli
                    export PATH=$(npm root -g)/.bin:$PATH  # Ensure 'ng' is available

                    echo "Cleaning cache and installing project dependencies..."
                    rm -rf node_modules package-lock.json || true
                    npm cache clean --force
                    npm install

                    echo "Angular CLI Version:" && ng version
                '''
            }
        }

        stage('Build Angular App') {
            steps {
                sh '''
                    echo "Ensuring Angular CLI is accessible..."
                    export PATH=$(npm root -g)/.bin:$PATH

                    echo "Building Angular project in production mode..."
                    ng build --configuration=production --output-path=dist/expense-tracker
                    ls -l dist/expense-tracker/
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Uncomment and adjust this section if you wish to deploy to a remote server.
                withCredentials([sshUserPrivateKey(credentialsId: 'EC2_SSH_KEY', keyFileVariable: 'SSH_KEY_PATH', usernameVariable: 'SSH_USER')]) {
                    sh '''
                        echo "Deploying to EC2..."
                        scp -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} -r ${BUILD_DIR}/* ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} 'sudo systemctl restart nginx'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful!"
        }
        failure {
            echo "Build failed. Check the logs for errors."
        }
    }
}

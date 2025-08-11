pipeline {
    agent any
    tools {
        jdk 'jdk21'
        nodejs 'node24'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token', // <-- Add your Jenkins GitHub credentials ID
                    url: 'https://github.com/Mahes-raj/Book-My-Show.git'
                sh 'ls -la'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd Book-My-Show
                ls -la
                if [ -f package.json ]; then
                    rm -rf node_modules package-lock.json
                    npm install
                else
                    echo "Error: package.json not found in Book-My-Show!"
                    exit 1
                fi
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        echo "Building Docker image..."
                        docker build --no-cache -t mahesraj/bms:latest -f Book-My-Show/Dockerfile Book-My-Show

                        echo "Pushing Docker image to registry..."
                        docker push mahesraj/bms:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                echo "Stopping and removing old container..."
                docker stop bms || true
                docker rm bms || true

                echo "Running new container on port 3000..."
                docker run -d --restart=always --name bms -p 3000:3000 mahesraj/bms:latest

                echo "Checking running containers..."
                docker ps -a

                echo "Fetching logs..."
                sleep 5
                docker logs bms
                '''
            }
        }
    }
}


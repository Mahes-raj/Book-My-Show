pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/KastroVKiran/Book-My-Show.git'
                sh 'ls -la'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                ls -la
                if [ -f package.json ]; then
                    rm -rf node_modules package-lock.json
                    npm install
                else
                    echo "Error: package.json not found in bookmyshow-app!"
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
                        docker build --no-cache -t kastrov/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app

                        echo "Pushing Docker image to registry..."
                        docker push kastrov/bms:latest
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
                docker run -d --restart=always --name bms -p 3000:3000 kastrov/bms:latest

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


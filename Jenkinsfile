pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'http://git.istad.co/cstad-ite-2nd-generation/fswd/alumni/alumni-ui.git', description: 'Repository URL')
        string(name: 'CONTAINER_NAME', defaultValue: 'nextjs-app', description: 'Docker Container Name')
        string(name: 'DEPLOY_PORT', defaultValue: '3000', description: 'Port for Deployment')
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the Next.js project from the repository
                    git branch: 'main', url: "${params.REPO_URL}"
                }
            }
        }
        
        stage('Build Next.js App') {
            steps {
                script {
                    // Install dependencies and build the Next.js project
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Define a simple Dockerfile for Next.js
                    def dockerfileContent = '''
                        FROM node:18-alpine
                        WORKDIR /app
                        COPY package.json package-lock.json ./
                        RUN npm install
                        COPY . .
                        RUN npm run build
                        EXPOSE 3000
                        CMD ["npm", "start"]
                    '''
                    writeFile file: 'Dockerfile', text: dockerfileContent
                    
                    // Build Docker image
                    sh "docker build -t ${params.CONTAINER_NAME}:latest ."
                }
            }
        }

        stage('Deploy with Docker') {
            steps {
                script {
                    // Stop and remove any running container with the same name
                    sh """
                        docker stop ${params.CONTAINER_NAME} || true
                        docker rm ${params.CONTAINER_NAME} || true
                    """
                    
                    // Run the container on the specified port
                    sh """
                        docker run -d -p ${params.DEPLOY_PORT}:3000 --name ${params.CONTAINER_NAME} ${params.CONTAINER_NAME}:latest
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}

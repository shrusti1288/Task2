# Task2
Create a Simple Jenkins Pipeline for CI/CD


Jenkinsfile

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/your-app-name"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building application...'
                // Add build steps here, e.g., npm install, mvn package, etc.
                sh 'echo "Build complete"'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t $DOCKER_IMAGE:latest ."
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push $DOCKER_IMAGE:latest"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // Replace with your deploy command (e.g., docker run on a remote server)
                sh "docker run -d -p 80:80 $DOCKER_IMAGE:latest"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}


Prerequisites

Jenkins with Docker installed and accessible.

DockerHub account with a repository.

Jenkins credentials with ID dockerhub-creds (username + password).

A valid Dockerfile in your project.

Project Structure Assumption

your-node-app/
│
├── Dockerfile
├── Jenkinsfile
├── package.json
└── src/


Node.js-Specific Jenkinsfile

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/your-node-app"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-username/your-node-repo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t $DOCKER_IMAGE:latest ."
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push $DOCKER_IMAGE:latest"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                // Optional: Stop and remove any existing container before running new
                sh '''
                    docker rm -f node-app || true
                    docker run -d --name node-app -p 3000:3000 $DOCKER_IMAGE:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Node.js CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}


Node.js Dockerfile Example

If you need one, here's a simple Dockerfile:

# Use Node.js LTS
FROM node:18

# Set working directory
WORKDIR /app

# Copy files and install dependencies
COPY package*.json ./
RUN npm install

# Copy app files
COPY . .

# Expose port and start app
EXPOSE 3000
CMD ["npm", "start"]


1. package.json

This sets up a basic Express server app:

{
  "name": "jenkins-node-app",
  "version": "1.0.0",
  "description": "Simple Node.js app for Jenkins CI/CD pipeline",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "test": "echo \"No tests yet\" && exit 0"
  },
  "dependencies": {
    "express": "^4.18.4"
  }
}

2. server.js

A basic Express server that listens on port 3000:

const express = require('express');
const app = express();

const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('Hello from Node.js CI/CD app using Jenkins and Docker!');
});

app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});


3. Dockerfile

Matches the one used in your Jenkinsfile:

FROM node:18

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]


How to Set Up in Jenkins:

1. Install Required Plugins:
Docker Pipeline
Git

2. Configure Credentials:
Go to Jenkins > Manage Jenkins > Credentials
Add DockerHub username/password with ID: dockerhub-creds

3. Create a New Pipeline Job:
Choose Pipeline project
Set Pipeline script from SCM and point to your Git repo with this code

4. Run the Pipeline



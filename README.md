# CI/CD Deployment Assignment

This repository contains the implementation of a CI/CD pipeline for deploying a Flask backend and Express frontend application on an Amazon EC2 instance using Jenkins.

## Project Overview

The project consists of two main components:
1. **Flask Backend** - REST API service running on port 5000
2. **Express Frontend** - Web application running on port 3000

### Repository Structure
- [Flask Backend](https://github.com/MukherjeePushpendu/flask-backend.git)
- [Express Frontend](https://github.com/MukherjeePushpendu/express-frontend.git)

## Live Demo

The application is deployed and accessible at: [http://54.175.149.120/](http://54.175.149.120/)

## Architecture

```
┌─────────────────────────────────────┐
│           EC2 Instance              │
│                                     │
│  ┌───────────┐    ┌───────────┐     │
│  │  Express  │    │   Flask   │     │
│  │ Frontend  │    │ Backend   │     │
│  │  Port 3000│    │ Port 5000 │     │
│  └───────────┘    └───────────┘     │
│                                     │
│  ┌─────────────────────────────┐    │
│  │         Jenkins            │    │
│  │     CI/CD Pipeline         │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │         Nginx              │    │
│  │     Reverse Proxy          │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

## CI/CD Pipeline Implementation

### Jenkins Setup

1. **Installation**
   ```bash
   # Update system
   sudo apt update
   sudo apt upgrade -y

   # Install Java
   sudo apt install openjdk-11-jdk -y

   # Add Jenkins repository
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null

   # Install Jenkins
   sudo apt update
   sudo apt install jenkins -y

   # Start Jenkins
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

2. **Required Plugins**
   - Git Integration
   - NodeJS Plugin
   - Python Plugin
   - Pipeline Plugin
   - GitHub Integration

### Pipeline Configuration

#### Flask Backend Pipeline (Jenkinsfile)
```groovy
pipeline {
    agent any
    
    environment {
        FLASK_APP = 'app.py'
        FLASK_ENV = 'production'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Python') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }
        
        stage('Test') {
            steps {
                sh '. venv/bin/activate && pytest'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'pm2 restart flask-backend || pm2 start app.py --name flask-backend'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

#### Express Frontend Pipeline (Jenkinsfile)
```groovy
pipeline {
    agent any
    
    environment {
        NODE_ENV = 'production'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Node.js') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'pm2 restart express-frontend || pm2 start npm --name express-frontend -- start'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

### GitHub Webhook Configuration

1. In GitHub repository settings:
   - Go to Settings > Webhooks
   - Add webhook
   - Payload URL: `http://<jenkins-url>/github-webhook/`
   - Content type: `application/json`
   - Select events: Push events

2. In Jenkins:
   - Configure job to build on GitHub webhook trigger
   - Add webhook URL to job configuration

## Setup Instructions

### Prerequisites
- AWS Account
- EC2 Instance (t2.micro recommended for free tier)
- Node.js (v14 or higher)
- Python 3.8 or higher
- Git
- Jenkins

### Backend Setup (Flask)
```bash
# Clone the repository
git clone https://github.com/MukherjeePushpendu/flask-backend.git

# Navigate to the directory
cd flask-backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Start the server
python app.py
```

### Frontend Setup (Express)
```bash
# Clone the repository
git clone https://github.com/MukherjeePushpendu/express-frontend.git

# Navigate to the directory
cd express-frontend

# Install dependencies
npm install

# Start the server
npm start
```

## Environment Variables

### Backend (.env)
```
PORT=5000
FLASK_ENV=development
```

### Frontend (.env)
```
PORT=3000
FLASK_API_URL=http://localhost:5000
```

## API Endpoints

### Backend (Flask)
- `GET /`: Welcome message
- `GET /api/status`: Health check endpoint

### Frontend (Express)
- `GET /`: Home page
- `GET /api/status`: Health check endpoint

## Deployment

The application is deployed on an Amazon EC2 instance with the following configuration:
- Instance Type: t2.micro
- OS: Ubuntu 22.04 LTS
- Public IP: 54.175.149.120

## Security

- CORS enabled for cross-origin requests
- Environment variables for sensitive data
- Nginx as reverse proxy for additional security
- Jenkins credentials for secure storage of sensitive data

## Monitoring

- PM2 for process management
- Nginx access logs
- Application logs in respective directories
- Jenkins build history and logs

## Screenshots

[Add your Jenkins pipeline screenshots here]

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License. 
# CI/CD Deployment Assignment Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Implementation Steps](#implementation-steps)
4. [Jenkins Pipeline Configuration](#jenkins-pipeline-configuration)
5. [Deployment Process](#deployment-process)
6. [Screenshots](#screenshots)

## Project Overview

This documentation details the implementation of a CI/CD pipeline for deploying a Flask backend and Express frontend application on an Amazon EC2 instance.

### Components
- Flask Backend (Port 5000)
- Express Frontend (Port 3000)
- Jenkins CI/CD Pipeline
- Nginx Reverse Proxy

## Architecture

The application is deployed on a single EC2 instance with the following components:

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

## Implementation Steps

### 1. EC2 Instance Setup
- Launch t2.micro instance (Ubuntu 22.04 LTS)
- Configure security groups for ports 22, 80, 3000, 5000
- Install required dependencies:
  ```bash
  # Update system
  sudo apt update
  sudo apt upgrade -y

  # Install Node.js
  curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
  sudo apt install -y nodejs

  # Install Python
  sudo apt install python3-pip python3-venv -y

  # Install PM2
  sudo npm install -g pm2

  # Install Nginx
  sudo apt install nginx -y
  ```

### 2. Jenkins Installation
```bash
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

### 3. Jenkins Configuration
1. Access Jenkins at `http://<ec2-public-ip>:8080`
2. Install required plugins:
   - Git Integration
   - NodeJS Plugin
   - Python Plugin
   - Pipeline Plugin
   - GitHub Integration

## Jenkins Pipeline Configuration

### Flask Backend Pipeline
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
}
```

### Express Frontend Pipeline
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
}
```

## Deployment Process

### 1. Application Deployment
1. Clone repositories:
   ```bash
   git clone https://github.com/MukherjeePushpendu/flask-backend.git
   git clone https://github.com/MukherjeePushpendu/express-frontend.git
   ```

2. Configure environment variables:
   - Backend (.env):
     ```
     PORT=5000
     FLASK_ENV=production
     ```
   - Frontend (.env):
     ```
     PORT=3000
     FLASK_API_URL=http://localhost:5000
     ```

3. Start applications:
   ```bash
   # Backend
   cd flask-backend
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   pm2 start app.py --name flask-backend

   # Frontend
   cd ../express-frontend
   npm install
   pm2 start npm --name express-frontend -- start
   ```

### 2. Nginx Configuration
```nginx
server {
    listen 80;
    server_name 54.175.149.120;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Screenshots

### Jenkins Pipeline
[Add Jenkins pipeline screenshots here]

### Application Deployment
[Add application deployment screenshots here]

### Nginx Configuration
[Add Nginx configuration screenshots here]

## Monitoring and Maintenance

### PM2 Process Management
```bash
# List all processes
pm2 list

# Monitor processes
pm2 monit

# View logs
pm2 logs

# Restart applications
pm2 restart all
```

### Nginx Logs
```bash
# Access logs
sudo tail -f /var/log/nginx/access.log

# Error logs
sudo tail -f /var/log/nginx/error.log
```

### Jenkins Logs
```bash
# View Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log
```

## Troubleshooting

### Common Issues and Solutions

1. **Application Not Starting**
   - Check PM2 logs: `pm2 logs`
   - Verify environment variables
   - Check port availability: `sudo netstat -tulpn | grep LISTEN`

2. **Nginx Issues**
   - Check Nginx status: `sudo systemctl status nginx`
   - Verify configuration: `sudo nginx -t`
   - Check error logs: `sudo tail -f /var/log/nginx/error.log`

3. **Jenkins Pipeline Failures**
   - Check build logs in Jenkins UI
   - Verify GitHub webhook configuration
   - Check Jenkins system logs

## Security Considerations

1. **Firewall Configuration**
   ```bash
   sudo ufw allow 22
   sudo ufw allow 80
   sudo ufw allow 3000
   sudo ufw allow 5000
   sudo ufw enable
   ```

2. **Environment Variables**
   - Store sensitive data in Jenkins credentials
   - Use .env files for local development
   - Never commit sensitive data to Git

3. **Nginx Security**
   - Enable HTTPS
   - Configure rate limiting
   - Set up proper CORS headers

## Future Improvements

1. **CI/CD Enhancements**
   - Add automated testing
   - Implement staging environment
   - Add deployment notifications

2. **Monitoring**
   - Set up application monitoring
   - Configure alerting
   - Implement log aggregation

3. **Security**
   - Implement SSL/TLS
   - Add WAF protection
   - Set up backup strategy 
# CI/CD Deployment Assignment

This repository contains the implementation of a CI/CD pipeline for deploying a Flask backend and Express frontend application on an Amazon EC2 instance.

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
┌─────────────────┐
│   EC2 Instance  │
│                 │
│  ┌───────────┐  │
│  │  Express  │  │
│  │ Frontend  │  │
│  │  Port 3000│  │
│  └───────────┘  │
│                 │
│  ┌───────────┐  │
│  │   Flask   │  │
│  │ Backend   │  │
│  │ Port 5000 │  │
│  └───────────┘  │
└─────────────────┘
```

## Setup Instructions

### Prerequisites
- AWS Account
- EC2 Instance (t2.micro recommended for free tier)
- Node.js (v14 or higher)
- Python 3.8 or higher
- Git

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

## CI/CD Pipeline

The project uses Jenkins for continuous integration and deployment. The pipeline includes:

1. **Build Stage**
   - Clone repositories
   - Install dependencies
   - Run tests

2. **Deploy Stage**
   - Deploy to EC2 instance
   - Configure Nginx as reverse proxy
   - Start services using PM2

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

## Monitoring

- PM2 for process management
- Nginx access logs
- Application logs in respective directories

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License. 
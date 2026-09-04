# DevOps Staging Environment Automation

An automated CI/CD pipeline that builds a Dockerized staging application and manages its lifecycle (start/stop) on a schedule — reducing staging infrastructure costs by only running the container during office hours.

## Project Goal

Staging environments are often left running 24/7 even though they're only used during working hours. This project automates:
- Building & deploying the staging app via a CI/CD pipeline (Jenkins)
- Scheduling the container to stop after hours and start again in the morning, using Ansible + Jenkins cron triggers

## Tech Stack

- **Jenkins** — CI/CD orchestration (Pipeline as Code)
- **Docker** — containerizes the staging application (nginx-based)
- **Ansible** — manages container start/stop lifecycle
- **Git/GitHub** — source control and pipeline trigger source
- **AWS EC2 (Amazon Linux 2023)** — hosts Jenkins, Docker, and the running container

## Architecture

```
GitHub Repo
     │
     ▼
Jenkins Pipeline (Checkout → Test → Docker Build → Deploy)
     │
     ▼
Docker Image (staging-app:latest)
     │
     ▼
Ansible Playbook (start.yml) ──► Container running on EC2
     │
     ▼
Scheduled Jenkins Jobs (cron) ──► stop.yml (end of day) / start.yml (morning)
```

## Repository Structure

```
.
├── Dockerfile        # Builds the staging-app image (nginx:alpine base)
├── inventory         # Ansible inventory (target host)
├── start.yml         # Ansible playbook to start the staging container
├── stop.yml          # Ansible playbook to stop the staging container
├── Jenkinsfile        # Main CI/CD pipeline (build + deploy)
└── README.md
```

## CI/CD Pipeline

The main Jenkins pipeline (triggered on demand or via SCM webhook) runs:

1. **Checkout** — pulls the latest code from the `master` branch
2. **Test** — sanity-checks that Ansible and Docker are available on the agent
3. **Docker Build** — builds the `staging-app:latest` image from the Dockerfile
4. **Deploy** — runs `ansible-playbook -i inventory start.yml` to launch/refresh the container

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Abdulrehman-Jamil-473/DevOps-Staging-Environment-Automation.git'
            }
        }
        stage('Test') {
            steps {
                sh 'ansible --version'
                sh 'docker --version'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t staging-app:latest .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'ansible-playbook -i inventory start.yml'
            }
        }
    }
}
```

Scheduled Jenkins Jobs (cron) ──► `stop.yml` (end of day) / `start.yml` (morning)

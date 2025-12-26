# LAB GUIDE: Git, Docker, and Jenkins CI/CD with Python FastAPI

## LAB ENVIRONMENT PREREQUISITES (COMMON FOR ALL LABS)

### Hardware & OS

- Laptop/Desktop (Windows / macOS / Linux)
- Minimum 8 GB RAM recommended
- Stable internet connection

### Software to Install

1. Git
2. Docker Desktop
3. Python 3.10 or above
4. Code Editor (VS Code recommended)
5. Jenkins (local installation using Docker)

# LAB 1: Git Commands and GitHub

## Overview

This lab introduces **version control using Git** and **remote collaboration using GitHub**.
Students will learn how to create repositories, track changes, and collaborate.

## Lab 1.1 – Git Installation and Configuration

### Objectives

- Install Git
- Configure username and email
- Verify Git setup

### Steps

1. Verify Git installation:

```bash
git --version
```

2. Configure Git user details:

```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```

3. Verify configuration:

```bash
git config --list
```

### Verification (Acceptance Criteria)

- `git --version` prints version number
- Username and email are visible in `git config --list`

## Lab 1.2 – Creating a Local Git Repository

### Objectives

- Initialize a Git repository
- Track files
- Perform first commit

### Steps

1. Create project directory:

```bash
mkdir git-lab
cd git-lab
```

2. Initialize repository:

```bash
git init
```

3. Create a file:

```bash
echo "My first Git project" > README.md
```

4. Check status:

```bash
git status
```

5. Stage and commit:

```bash
git add README.md
git commit -m "Initial commit"
```

### Verification

- `git status` shows clean working tree
- `git log --oneline` shows commit

## Lab 1.3 – Working with Branches

### Objectives

- Create branches
- Switch branches
- Merge changes

### Steps

1. Create branch:

```bash
git branch feature-1
```

2. Switch branch:

```bash
git checkout feature-1
```

3. Modify file:

```bash
echo "Feature work" >> README.md
git add .
git commit -m "Added feature"
```

4. Merge back to main:

```bash
git checkout main
git merge feature-1
```

### Verification

- `git branch` shows branches
- README.md contains merged content

## Lab 1.4 – GitHub Remote Repository

### Objectives

- Create GitHub repository
- Push local code
- Pull changes

### Steps

1. Create repository in GitHub (without README)

2. Add remote:

```bash
git remote add origin https://github.com/<username>/git-lab.git
```

3. Push code:

```bash
git push -u origin main
```

4. Make a change in GitHub UI and pull:

```bash
git pull origin main
```

### Verification

- Code visible on GitHub
- Local repo updated after pull

# LAB 2: Docker – From Basics to Docker Hub

## Overview

This lab introduces **containerization using Docker** and walks through building and publishing images.

## Lab 2.1 – Docker Installation and Basics

### Objectives

- Verify Docker installation
- Run first container

### Steps

1. Verify Docker:

```bash
docker --version
docker info
```

2. Run hello-world:

```bash
docker run hello-world
```

### Verification

- Docker version displayed
- Success message from hello-world container

## Lab 2.2 – Running Containers

### Objectives

- Pull images
- Run containers
- Stop and remove containers

### Steps

1. Pull image:

```bash
docker pull nginx
```

2. Run container:

```bash
docker run -d -p 8080:80 nginx
```

3. Verify in browser:

```
http://localhost:8080
```

4. Stop and remove:

```bash
docker ps
docker stop <container_id>
docker rm <container_id>
```

### Verification

- Nginx page loads
- Container stops successfully

## Lab 2.3 – Dockerizing a Python FastAPI Application

### Objectives

- Create FastAPI app
- Build Docker image
- Run containerized app

### Steps

1. Create project:

```bash
mkdir fastapi-app
cd fastapi-app
```

2. Create `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI"}
```

3. Create `requirements.txt`:

```
fastapi
uvicorn
```

4. Create `Dockerfile`:

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

5. Build image:

```bash
docker build -t fastapi-demo .
```

6. Run container:

```bash
docker run -d -p 8000:8000 fastapi-demo
```

### Verification

- API accessible at `http://localhost:8000`
- JSON response returned

## Lab 2.4 – Push Image to Docker Hub

### Objectives

- Create Docker Hub account
- Push custom image

### Steps

1. Login:

```bash
docker login
```

2. Tag image:

```bash
docker tag fastapi-demo <dockerhub-username>/fastapi-demo:v1
```

3. Push image:

```bash
docker push <dockerhub-username>/fastapi-demo:v1
```

### Verification

- Image visible in Docker Hub repository
- Pull works on another machine

# LAB 3: Jenkins CI/CD Pipeline with FastAPI

## Overview

This lab demonstrates **CI/CD using Jenkins** by building and deploying a Dockerized FastAPI app.

## Lab 3.1 – Jenkins Setup Using Docker

### Objectives

- Run Jenkins
- Access UI
- Install required plugins

### Steps

1. Run Jenkins:

```bash
docker run -d \
-p 8080:8080 \
-p 50000:50000 \
-v jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts
```

2. Access:

```
http://localhost:8080
```

3. Unlock Jenkins:

```bash
docker exec <jenkins_container> cat /var/jenkins_home/secrets/initialAdminPassword
```

4. Install suggested plugins

### Verification

- Jenkins dashboard accessible
- Admin user created

## Lab 3.2 – Create Jenkins Pipeline Project

### Objectives

- Create pipeline job
- Connect GitHub repository

### Steps

1. Create **New Item**
2. Select **Pipeline**
3. Configure:

   - SCM: Git
   - Repository URL
   - Branch: main

## Lab 3.3 – Jenkinsfile for CI/CD

### Objectives

- Automate build and push
- Validate pipeline execution

### Jenkinsfile

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<username>/fastapi-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t fastapi-jenkins .'
            }
        }
        stage('Tag and Push') {
            steps {
                sh '''
                docker tag fastapi-jenkins <dockerhub-username>/fastapi-jenkins:latest
                docker push <dockerhub-username>/fastapi-jenkins:latest
                '''
            }
        }
    }
}
```

### Verification (Acceptance Criteria)

- Jenkins job runs successfully
- Docker image updated in Docker Hub
- Console log shows no errors

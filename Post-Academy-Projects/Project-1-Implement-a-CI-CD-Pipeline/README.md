# Project-1-Implement-a-CI-CD-Pipeline

## Create Jenkins Server

1. **Setup an EC2 Instance on AWS**

   - Launch an EC2 instance on AWS.
   - Configure a custom security group with port 8080 open.

2. **SSH into the EC2 Instance**

   - Use a terminal (e.g., Gitbash) to SSH into your EC2 instance.

3. **Install Jenkins**

    ```bash
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
     
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null

   sudo apt-get install jenkins
    ```

4. **Install Java**

   ```bash
   sudo apt update -y
   sudo apt upgrade -y
   sudo apt install fontconfig openjdk-17-jre
   java -version
   ```

5. **Start Jenkins**

   ```bash
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   ```

## Set up Dockerfile

1. **Create Dockerfile**

   In the app repo create the Dockerfile.

   ```dockerfile
   # syntax=docker/dockerfile:1
    
   FROM node:20-alpine
    
   WORKDIR /app
    
   COPY package*.json ./
    
   COPY . .
    
   RUN npm install
    
   USER node
    
   EXPOSE 3000
    
   CMD node app.js

   ```

2. **Test the Dockerfile**

   - Build the image `docker build -t my-node-app .`
   - Run the container `docker run -p 3000:3000 my-node-app`
   - Navigate to <http://localhost:3000> and confirm it's working.
     ![alt text](img/image.png)
   - Check that the home page and other routes (e.g., /fibonacci/10) work correctly.
     ![alt text](img/image-1.png)

3. **Tag and push the Docker Image**

   - Tag the Docker image: `docker tag my-node-app:latest hussainajhar32/project-1-sparta-app:latest`
   - Push the Docker Image: `docker push hussainajhar32/project-1-sparta-app:latest`
   - Verify that the Docker image is successfully pushed to Docker Hub.
     ![alt text](img/image-2.png)

## Kubernetes Deployment and Service Setup

1. **Create Deployment and Service files (and Ingress File for production):**

   - Create `deployment.yaml`, `service.yaml`, and optionally `ingress.yaml` for production.

   - Apply the files to Kubernetes:

   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   # Apply ingress if using
   kubectl apply -f ingress.yaml
   ```

2. **Verify the Deployment:**

   - Verify that pods are running and the service is created: `kubectl get pods`, `kubectl get services`
     ![alt text](img/image-3.png)
   - Verify the application is running on `localhost:30001`
     ![alt text](img/image-4.png)

## Configure Jenkins Job

1. **Configure the webhook on GitHub.**
   - Go onto the github repo
   - Go to settings and click on webhooks
   - Add Jenkins payload url `http://44.223.26.166:8080/github-webhook/`

2. **Create an item as a pipeline in Jenkins.**
   - In Jenkins server create a new job
   - Select Pipeline
   - Configure appropriately

3. **Add Jenkinsfile as the pipeline script.**

   ```groovy
   pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'project-1-sparta-app:latest'
        DOCKER_REGISTRY = 'hussainajhar32/project-1-sparta-app'
        DOCKER_REGISTRY_CREDENTIALS = 'ajhar-dockerhub-credentials'
        GIT_CREDENTIALS = 'ajhar-github-credentials'
        KUBERNETES_CREDENTIALS = 'ajhar-kube-config'
        KUBERNETES_DEPLOYMENT_NAME = 'project-1-sparta-app-deployment'
        KUBERNETES_SERVICE_NAME = 'project-1-sparta-app-svc'
        DEV_BRANCH = 'dev'
        MAIN_BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.DEV_BRANCH}", credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/your-github-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_REGISTRY}:${env.BUILD_ID}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Replace with your testing commands
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_REGISTRY_CREDENTIALS}") {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Merge to Main') {
            steps {
                script {
                    sh 'git config --global user.email "you@example.com"'
                    sh 'git config --global user.name "Your Name"'
                    sh """
                    git checkout ${MAIN_BRANCH}
                    git merge ${DEV_BRANCH}
                    git push origin ${MAIN_BRANCH}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${KUBERNETES_CREDENTIALS}"]) {
                        sh """
                        kubectl set image deployment/${KUBERNETES_DEPLOYMENT_NAME} ${DOCKER_IMAGE}=${DOCKER_REGISTRY}:latest
                        kubectl rollout status deployment/${KUBERNETES_DEPLOYMENT_NAME}
                        """
                    }
                }
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

4. **Add the credentials for github, docker and kubernetes.**
   - For Github credentials, create `Secret text` and add a personal access token
   - For Dockerhub, click on `Username with password` and add your details
   - For Kubernetes, click on `Secret file` and then upload the file `~/.kube/config`.
   ![alt text](img/image-5.png)

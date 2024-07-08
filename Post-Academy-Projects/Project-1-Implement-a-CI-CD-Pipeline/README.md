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
echo "
```

4. **Install Java**

```bash
sudo apt update
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
- Navigate to http://localhost:3000 and confirm it's working.
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

1. **Verify the Deployment:**
- Verify that pods are running and the service is created: `kubectl get pods`, `kubectl get services`
  ![alt text](img/image-3.png)
- Verify the application is running on `localhost:30001`
  ![alt text](img/image-4.png)

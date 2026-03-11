# Jenkins CI/CD Pipeline (SonarQube, Docker, ArgoCD and Kubernetes)

March 11, 2026

This project demonstrates an end-to-end CI/CD and GitOps workflow for deploying a Spring Boot application using Jenkins, SonarQube, Docker, ArgoCD, and Kubernetes.

The pipeline automatically builds the application, performs static code analysis, packages the application into a Docker image, pushes the image to Docker Hub, updates the Kubernetes deployment manifest, and allows ArgoCD to synchronize the change into a Kubernetes cluster.

The infrastructure combines cloud and local environments:

- **AWS EC2** hosts Jenkins and SonarQube for CI and code analysis.
- **Minikube** (local Kubernetes cluster) runs the application workloads.
- **ArgoCD** continuously monitors the Git repository and deploys updates to Kubernetes.

This architecture follows a GitOps model, where the Kubernetes cluster state is driven by version-controlled manifests stored in Git.

Link to Jenkins Implementation by Abhishek Veeramalla: https://www.youtube.com/watch?v=JGQI5pkK82w

## Architecture Diagram

<img width="1032" height="364" alt="Jenkins CI-CD project" src="https://github.com/user-attachments/assets/2b0abfa7-162b-4907-9c6c-8afe31dd9678" />

## Key Technologies

- **CI/CD -** Jenkins Pipeline, Docker Agent
- **Code Quality -** SonarQube, Maven Sonar Scanner
- **Containerization -** Docker Hub Image Registry
- **Kubernetes & GitOps -** Minikube, ArgoCD, ArgoCD Operator, Kubernetes Deployments and Services
- **Cloud & Infrastructure -** AWS EC2, Linux (Ubuntu)
- **Build Tools -** Maven, Spring Boot

## Pipeline Workflow

**1. Build and Test**
The Spring Boot project is compiled and packaged using Maven. This generates the application JAR file.
```
mvn clean package
```

<img width="1015" height="137" alt="image" src="https://github.com/user-attachments/assets/6d5f247b-e7aa-4eea-a9d4-fca80753cdfa" />

**2. Static Code Analysis**
The pipeline sends the project to SonarQube for static code analysis. SonarQube evaluates code quality, detects vulnerabilities, and provides metrics on technical debt.
```
mvn sonar:sonar
```

<img width="994" height="224" alt="image" src="https://github.com/user-attachments/assets/e52868c3-7adf-42ce-81a2-d42236c0dfc6" />

**3. Build Docker Image**
The packaged application is containerized using a Dockerfile. Each Jenkins build generates a uniquely tagged image.
```
yitzamm/jenkins-pipeline:<build-number>
```

**4. Push Image to Docker Hub**
The pipeline authenticates with Docker Hub and pushes the image to the registry. This allows Kubernetes clusters to pull the image for deployment.

<img width="922" height="228" alt="image" src="https://github.com/user-attachments/assets/d6f0a57c-d490-4b24-b918-14c041901fd9" />

**5. Update Kubernetes Deployment Manifest**
Jenkins updates the Kubernetes deployment file by replacing a placeholder tag with the new build number. Before pipeline:
```
image: yitzamm/jenkins-pipeline:replaceImageTag
```
After:
```
image: yitzamm/jenkins-pipeline:4
```

**6. ArgoCD GitOps Deployment**
ArgoCD continuously monitors the Git repository. When it detects the manifest update:

- ArgoCD pulls the updated manifest
- Compares desired vs actual cluster state
- Deploys the updated container image to Kubernetes

<img width="1496" height="508" alt="image" src="https://github.com/user-attachments/assets/f74364d9-8ce6-49bc-9eca-aff41fd79fc0" />

**7. Kubernetes Deployment**
The application runs in Kubernetes pods managed by a Deployment controller.
```
replicas: 2
```
Kubernetes ensures:

- High availability
- Self-healing pods
- Load balancing across replicas

<img width="992" height="344" alt="image" src="https://github.com/user-attachments/assets/9f9863e6-6679-4207-bbc4-d83fa2a08c58" />

The application is exposed using a NodePort service.
```
spring-boot-app-service   NodePort   80:32382/TCP
```

<img width="1908" height="554" alt="image" src="https://github.com/user-attachments/assets/3eba3f4b-05e4-4907-a309-a368167d449e" />

## Suggested Improvement: Automatic Pipeline Trigger
A recommended improvement would be configuring GitHub Webhooks so Jenkins automatically triggers the pipeline whenever changes are pushed to the repository. To enable it we would need to:

1. Configure a GitHub Webhook pointing to the Jenkins server:
```
http://<jenkins-server>/github-webhook/
```

2. In Jenkins pipeline configuration, enable:
```
GitHub hook trigger for GITScm polling
```

3. Ensure Jenkins is reachable from GitHub (public IP or reverse proxy).

Other nice enhancements could be:

- Implement Ingress + NGINX instead of NodePort
- Add Helm charts for Kubernetes deployments
- Introduce automated rollback strategies
- Add monitoring with Prometheus and Grafana
- Add unit or integration tests, and/or implement container security scanning

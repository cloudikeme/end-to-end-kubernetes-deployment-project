# E-commerce Deployment with Kubernetes: A Hands-on Project 🚀

## Introduction 

Launching an e-commerce website is an exciting endeavor, but ensuring it can handle the demands of the modern web is no small feat.  Imagine your website going viral (wouldn't that be amazing?!).  How would you handle the surge in traffic? How can you update your application without interrupting your users' shopping spree?  

These are some of the challenges that containerization and Kubernetes (K8s) are designed to solve.

### The Challenges of Modern Web Deployment

- **Scalability:**  Your application needs to adapt to varying traffic levels, from a trickle of visitors to a flood of eager shoppers. 
- **Consistency:** You need your application to run smoothly and reliably across different environments (development, testing, production) without compatibility issues.
- **Availability:**  Updates and maintenance should be seamless, with zero downtime, so your customers can shop uninterrupted. 

### Containerization and Kubernetes to the Rescue!

**Docker** is our containerization hero, packaging our application and its dependencies into a portable unit that runs consistently anywhere.  Think of it as a self-contained capsule for your code!

**Kubernetes** acts as the conductor, orchestrating the deployment, scaling, and management of our containerized application.  Here's how it tackles our challenges:

- **Dynamic Scaling:** Kubernetes automatically adjusts resources based on real-time demand.  More traffic? No problem, Kubernetes spins up more instances of your application to handle the load!
- **Self-Healing:**  If a container crashes, Kubernetes automatically restarts it on a healthy node, ensuring continuous availability.
- **Seamless Updates & Rollbacks:**  Kubernetes allows you to deploy updates gradually, with zero downtime. And if something goes wrong, rolling back to a previous version is quick and easy. 

### Why This Project Matters

This project will guide you through deploying a real-world e-commerce application using Docker and Kubernetes. You'll gain practical experience with:

- **Containerizing an application with Docker**
- **Deploying and managing applications on Kubernetes**
- **Implementing scalability and self-healing**
- **Performing seamless updates and rollbacks**

By the end, you'll have a solid foundation in modern DevOps practices and the confidence to deploy and manage your applications like a pro! 

1. [Prerequisites & Setup](01-prerequisites-setup.md)
2. [Containerize App & Database](02-containerize-your-e-commerce-website-and-database.md)
3. [KinD Kubernetes Cluster Setup](03-setup-kubernetes-on-local-kind-cluster.md)
4. [Deploy App to Kubernetes](04-deploy-to-kubernetes.md)
5. [Exposing the Application to the Outside World](05-expose-your-website.md)
6. [Configuration Management](06-implement-configuration-management.md)
7. [Scaling the Application](07-scale-your-application.md)
8. [Implement a Rolling Update](08-perform-a-rolling-update.md)
9. [Roll Back Deployment](09-roll-back-a-deployment.md)
10. [Autoscale the Application](10-autoscale-my-application.md)
11. [Liveness & Readiness Probes](11-implement-liveness-readiness-probes.md)
12. [ConfigMaps & Secrets](11-implement-liveness-readiness-probes.md)
13. [Helm Chart Setup](13-package-everything-in-helm.md)
14. [Setup Persistent Storage aka PersistentVolumeClaim](14-implement-persistent-storage.md)
15. [CI/CD Pipeline Basic Setup](15-implement-basic-ci-cd-pipeline.md)

[Next](01-prerequisites-setup.md)






### 02

lets say you starting this afresh

#### create a new directory for your kubernetes project and change directory into it

```bash
mkdir k8s-app-project
cd k8s-app-project
```
#### create the app directory, which would house all the source code and files for the app and change directory into it

```bash
mkdir app
cd app
```
Get all your code files into this folder copy all the code files into this folder

#### Create a Dockerfile

Now, Create a Dockerfile that instructs Docker to: 

* Use php:7.4-apache as the base image.
* Install mysqli extension for PHP.
* Copy the application source code to /var/www/html/.
* Update database connection strings to point to a Kubernetes service named mysql-service.
* Expose port 80 to allow traffic to the web server.

```bash
touch Dockerfile
```

Put this in your Dockerfile

```dockerfile
FROM php:7.4-apache

RUN docker-php-ext-install mysqli

COPY . /var/www/html/

ENV DB_HOST=mysql-service \
    DB_USER=ecomuser \
    DB_PASSWORD=ecompassword \
    DB_NAME=ecomdb

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Add this line for debugging
RUN echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

EXPOSE 80

# Add this to print environment variables when the container starts
## CMD echo "DB_HOST: $DB_HOST" && \
    ## echo "DB_USER: $DB_USER" && \
    # echo "DB_PASSWORD: $DB_PASSWORD" && \
    # echo "DB_NAME: $DB_NAME" && \
    # apache2-foreground
```

You now have a dockerfile ready to be built into a docker image.

#### Build and Push the docker Image to docker hub

**Build the Docker image from the directory containing the Dockerfile:**

   ```
   docker build -t cloudikeme/ecom-web:v0.1 .
   ```

**You can then run the container locally for testing:**

   `docker run -p 8080:80 cloudikeme/ecom-web:v0.1`

   This will map port 8080 on your host to port 80 in the container.

**Push it to Docker Hub with :**

`docker push cloudikeme/ecom-web:v0.1`

Outcome: Your web application Docker image is now available on Docker Hub.

### 03

#### Setup KinD Cluster

**Create a single node Kubernetes cluster using KinD**

```bash
kind create cluster --name=web-app-project
```

confirm your node and cluster is running:

```bash
kubectl get nodes
```

#### Deploy Website to Kubernetes

Create directory 'manifests'

```bash
mkdir manifests
cd manifests
```

Create first manifest : website_deployment.yaml

Since we having 

```bash
touch website-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-website
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ecommerce-website
  template:
    metadata:
      labels:
        app: ecommerce-website
    spec:
      containers:
      - name: ecommerce-website
        image: cloudikeme/ecom-web:v1
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: mysql-service
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: password
        - name: DB_NAME
          value: ecommerce_db
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-website-service
spec:
  selector:
    app: ecommerce-website
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```



























































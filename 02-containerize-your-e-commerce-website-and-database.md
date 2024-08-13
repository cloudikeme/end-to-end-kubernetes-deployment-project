# Step 2: Containerize Your E-Commerce Website and Database

### Web Application Containerization

#### Go into the base of the directory

#### Create a Dockerfile 

Create a Dockerfile for your e-commerce application based on the requirements specified. Here's the Dockerfile content:

```dockerfile
# Use PHP 7.4 with Apache as the base image
FROM php:7.4-apache

# Install mysqli extension
RUN docker-php-ext-install mysqli

# Copy the application source code to the container
COPY . /var/www/html/

# Update database connection strings
# Assuming your connection string is in a file named config.php
RUN sed -i 's/localhost/mysql-service/g' /var/www/html/config.php

# Apache already exposes port 80, but we'll make it explicit
EXPOSE 80

# The default command from the base image will start Apache
```

Let's break down this Dockerfile:

1. `FROM php:7.4-apache`: This uses the official PHP 7.4 image with Apache as the base.

2. `RUN docker-php-ext-install mysqli`: This installs the mysqli extension for PHP.

3. `COPY . /var/www/html/`: This copies all files from the current directory (where the Dockerfile is located) to the Apache web root in the container.

4. `RUN sed -i 's/localhost/mysql-service/g' /var/www/html/config.php`: This command updates the database connection string in your config.php file. It replaces 'localhost' with 'mysql-service'. Make sure your config file is named config.php and contains the database connection string.

5. `EXPOSE 80`: This explicitly documents that the container will listen on port 80.

ToDo:

1. Save it as `Dockerfile` (no file extension) in the root directory of your e-commerce application.

2. Make sure your database connection configuration is in a file named `config.php` in the same directory. If it's named differently or located elsewhere, adjust the `sed` command accordingly.

#### Build and Push the docker Image to docker hub

3. Build the Docker image from the directory containing the Dockerfile:
4. 
   ```
   docker build -t my-ecommerce-app .
   ```

5. You can then run the container locally for testing:

   ```
   docker run -p 8080:80 my-ecommerce-app
   ```

   This will map port 8080 on your host to port 80 in the container.

Push it to Docker Hub with :

`docker push yourdockerhubusername/ecom-web:v1.`

Outcome: Your web application Docker image is now available on Docker Hub.

### Database Containerization

#### **Prepare the database initialization script (db-load-script.sql)**

Create a file named `db-load-script.sql` with the following content:

```sql
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- Create your tables here
CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    description TEXT
);

-- Insert some sample data
INSERT INTO products (name, price, description) VALUES
('Product 1', 19.99, 'Description for Product 1'),
('Product 2', 29.99, 'Description for Product 2'),
('Product 3', 39.99, 'Description for Product 3');

-- Add more tables and data as needed
```




Remember, when deploying to Kubernetes, you'll need to create a service named `mysql-service` that points to your MySQL database, so that the application can connect to it using this hostname.

[Next](03-setup-kubernetes-on-local-kind-cluster.md)
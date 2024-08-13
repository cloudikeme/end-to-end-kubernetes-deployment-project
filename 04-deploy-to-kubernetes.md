

Let's break this down into steps:

1. Create the website deployment YAML file
2. Prepare the database initialization script
3. Create a ConfigMap for the database script
4. Create a MariaDB deployment and service

Let's start with the website deployment:

1. **website-deployment.yaml**

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


3. **Create a ConfigMap for the database script**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-initdb-config
data:
  init.sql: |
    CREATE DATABASE IF NOT EXISTS ecommerce_db;
    USE ecommerce_db;

    CREATE TABLE IF NOT EXISTS products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        price DECIMAL(10, 2) NOT NULL,
        description TEXT
    );

    INSERT INTO products (name, price, description) VALUES
    ('Product 1', 19.99, 'Description for Product 1'),
    ('Product 2', 29.99, 'Description for Product 2'),
    ('Product 3', 39.99, 'Description for Product 3');
```

4. **Create MariaDB deployment and service (mariadb-deployment.yaml)**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
type: Opaque
stringData:
  username: dbuser
  password: dbpassword
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mariadb:10.5
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: password
        - name: MYSQL_DATABASE
          value: ecommerce_db
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: password
        volumeMounts:
        - name: mysql-initdb
          mountPath: /docker-entrypoint-initdb.d
      volumes:
      - name: mysql-initdb
        configMap:
          name: mysql-initdb-config
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

To deploy your e-commerce application to Kubernetes:

1. Apply the ConfigMap:
   ```
   kubectl apply -f mysql-initdb-config.yaml
   ```

2. Deploy MariaDB:
   ```
   kubectl apply -f mariadb-deployment.yaml
   ```

3. Deploy the website:
   ```
   kubectl apply -f website-deployment.yaml
   ```

After applying these configurations:

- The MariaDB database will be running with the initialization script applied.
- The e-commerce website will be deployed with two replicas.
- The website will be accessible through a LoadBalancer service.

Make sure to adjust the database connection settings in your PHP application to use the environment variables:

```php
$host = getenv('DB_HOST');
$user = getenv('DB_USER');
$password = getenv('DB_PASSWORD');
$database = getenv('DB_NAME');

$conn = new mysqli($host, $user, $password, $database);
```

This setup provides a scalable and maintainable deployment of your e-commerce application on Kubernetes, with the database properly initialized and the website connected to it.

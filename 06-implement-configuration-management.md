# Implement Configuration Management

### **Modify the Web Application**

First, let's assume you've modified your PHP application to check for an environment variable `FEATURE_DARK_MODE`. Here's a simple example of how this might look in your PHP code:

```php
<?php
$darkModeEnabled = getenv('FEATURE_DARK_MODE') === 'true';
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-commerce Store</title>
    <style>
        body {
            background-color: <?php echo $darkModeEnabled ? '#333' : '#fff'; ?>;
            color: <?php echo $darkModeEnabled ? '#fff' : '#333'; ?>;
        }
        /* Add more styles as needed */
    </style>
</head>
<body>
    <!-- Your e-commerce website content -->
</body>
</html>
```

### **Create a ConfigMap**

Create a file named `feature-toggle-configmap.yaml` with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-toggle-config
data:
  FEATURE_DARK_MODE: "true"
```

### **Deploy ConfigMap**

Apply the ConfigMap to your Kubernetes cluster:

```bash
kubectl apply -f feature-toggle-configmap.yaml
```

### **Update Deployment**

Modify your `website-deployment.yaml` to include the environment variable from the ConfigMap. Here's an updated version of the deployment:

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
        - name: FEATURE_DARK_MODE
          valueFrom:
            configMapKeyRef:
              name: feature-toggle-config
              key: FEATURE_DARK_MODE
```

### **Apply the Updated Deployment**

Apply the updated deployment to your Kubernetes cluster:

```bash
kubectl apply -f website-deployment.yaml
```

### **Outcome and Extras**

Your website should now render in dark mode, demonstrating how ConfigMaps can manage application features.

#### To verify:
1. Check if the pods are running with the updated configuration:
   ```
   kubectl get pods
   ```

2. You can check the environment variables of a pod to ensure the FEATURE_DARK_MODE is set correctly:
   ```
   kubectl exec <pod-name> -- printenv | grep FEATURE_DARK_MODE
   ```

3. Access your website through the LoadBalancer service you set up earlier. It should now be in dark mode.

#### To toggle the feature off:
1. Update the ConfigMap:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: feature-toggle-config
   data:
     FEATURE_DARK_MODE: "false"
   ```

2. Apply the updated ConfigMap:
   ```
   kubectl apply -f feature-toggle-configmap.yaml
   ```

3. Restart the deployment to pick up the new configuration:
   ```
   kubectl rollout restart deployment ecommerce-website
   ```

This implementation demonstrates how you can use Kubernetes ConfigMaps to manage feature toggles in your application. It allows you to change application behavior without modifying the code or rebuilding the Docker image, providing a flexible way to manage features in your e-commerce platform
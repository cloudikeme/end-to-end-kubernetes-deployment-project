# Step 12: Utilize ConfigMaps and Secrets


In this step, i will utilize ConfigMaps and Secrets to manage your database connection string and feature toggles securely. This approach follows best practices for configuration and secret management in Kubernetes.

### **Create Secret and ConfigMap**

First, let's create a Secret for the database credentials and a ConfigMap for feature toggles.

#### Create a Secret for database credentials:

```bash
kubectl create secret generic db-credentials \
    --from-literal=DB_USER=myuser \
    --from-literal=DB_PASSWORD=mypassword \
    --from-literal=DB_HOST=mysql-service \
    --from-literal=DB_NAME=ecommerce_db
```

#### Create a ConfigMap for feature toggles:

Create a file named `feature-toggles.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-toggles
data:
  FEATURE_DARK_MODE: "true"
  FEATURE_NEW_PRODUCT_PAGE: "false"
```

Apply the ConfigMap:

```bash
kubectl apply -f feature-toggles.yaml
```

### **Update Deployment**

Now, let's update the `website-deployment.yaml` to use these Secret and ConfigMap:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-website
spec:
  replicas: 3
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
        image: yourdockerhubusername/ecom-web:v2
        ports:
        - containerPort: 80
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_PASSWORD
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_HOST
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_NAME
        envFrom:
        - configMapRef:
            name: feature-toggles
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```

#### **Apply the Updated Deployment**

Apply the updated deployment:

```bash
kubectl apply -f website-deployment.yaml
```

#### **Verify the Configuration**

To verify that the environment variables are correctly set, you can exec into one of the pods and check:

```bash
kubectl exec -it $(kubectl get pods -l app=ecommerce-website -o jsonpath="{.items[0].metadata.name}") -- env | grep -E 'DB_|FEATURE_'
```

You should see the database credentials and feature toggle values.

#### **Update Application Code**

Ensure your application code is updated to use these environment variables. For example, in PHP:

```php
<?php
$db_user = getenv('DB_USER');
$db_password = getenv('DB_PASSWORD');
$db_host = getenv('DB_HOST');
$db_name = getenv('DB_NAME');

$dark_mode_enabled = getenv('FEATURE_DARK_MODE') === 'true';
$new_product_page_enabled = getenv('FEATURE_NEW_PRODUCT_PAGE') === 'true';

// Use these variables in your application logic
?>
```

### **Outcome**:
Your application configuration is now externalized and securely managed. Sensitive data (database credentials) are stored in a Secret, while non-sensitive configuration (feature toggles) are stored in a ConfigMap. This demonstrates best practices in configuration and secret management in Kubernetes.

**Additional Considerations**:

1. **Secret Encryption**: Ensure that encryption at rest is enabled for my Kubernetes secrets. This is often a cluster-wide setting.

2. **Rotation**: Implement a process for rotating secrets regularly. You can update the Secret in Kubernetes, and the new values will be available to your pods after a short delay.

3. **Version Control**: While you should never store raw secrets in version control, you can (and should) version control your ConfigMaps. For Secrets, consider using a template with placeholders.

4. **Access Control**: Use Kubernetes RBAC to control which pods and users can access which Secrets and ConfigMaps.

5. **External Secret Management**: For production environments, consider using an external secret management system like HashiCorp Vault or AWS Secrets Manager, integrated with Kubernetes.

6. **Environment-Specific Configs**: You might want to create different ConfigMaps for different environments (dev, staging, prod).

7. **Volumes**: For larger configurations or if you need file-based config, you can mount ConfigMaps and Secrets as volumes instead of using them as environment variables.

Here's a summary of the key commands i used:

```bash
# Create a Secret
kubectl create secret generic db-credentials --from-literal=...

# Apply a ConfigMap
kubectl apply -f feature-toggles.yaml

# Apply the updated deployment
kubectl apply -f website-deployment.yaml

# Verify environment variables
kubectl exec -it $(kubectl get pods -l app=ecommerce-website -o jsonpath="{.items[0].metadata.name}") -- env | grep -E 'DB_|FEATURE_'
```

By implementing these practices, i've significantly improved the security and flexibility of your e-commerce application's configuration. I can now easily update configuration without rebuilding my application, and sensitive data is kept separate and secure.
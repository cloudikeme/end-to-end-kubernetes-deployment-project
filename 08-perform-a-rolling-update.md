# Step 8: Perform a Rolling Update

In this step you would be performing a rolling update to add a promotional banner to your e-commerce website. This will demonstrate how Kubernetes can update your application with zero downtime.

### **Update Application**

First, let's assume you've modified your PHP application to include the promotional banner. Here's a simple example of how this might look in your PHP code:

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-commerce Store</title>
    <style>
        .promo-banner {
            background-color: #ffcc00;
            color: #333;
            text-align: center;
            padding: 10px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="promo-banner">
        🎉 Special Offer: 20% off all items for our marketing campaign! Use code: PROMO20 🎉
    </div>
    <!-- Rest of your e-commerce website content -->
</body>
</html>
```

### **Build and Push New Image**

Now, let's build and push the updated Docker image. Replace `yourdockerhubusername` with your actual Docker Hub username:

```bash
docker build -t yourdockerhubusername/ecom-web:v2 .
docker push yourdockerhubusername/ecom-web:v2
```

### **Rolling Update**

Update your `website-deployment.yaml` file with the new image version. Here's the relevant part that needs to be changed:

```yaml
spec:
  containers:
  - name: ecommerce-website
    image: yourdockerhubusername/ecom-web:v2  # Updated image version
```

Now, apply the changes:

```bash
kubectl apply -f website-deployment.yaml
```

### **Monitor Update**

To watch the rolling update process:

```bash
kubectl rollout status deployment/ecommerce-website
```

You should see output similar to:

```
Waiting for rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for rollout to finish: 6 out of 6 new replicas have been updated...
deployment "ecommerce-website" successfully rolled out
```

### **Verify the Update**

To confirm that all pods are running the new version:

```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

You should see all pods using the `:v2` image.

### **Check the Website**

Access your website through the LoadBalancer service. You should now see the promotional banner at the top of the page.

### **Outcome**:
Your website has been updated with the new promotional banner without any downtime. Kubernetes performed a rolling update, gradually replacing old pods with new ones, ensuring that the service remained available throughout the process.

**Additional Considerations**:

1. **Rollback Plan**: If you notice any issues with the new version, you can quickly rollback to the previous version:

   ```bash
   kubectl rollout undo deployment/ecommerce-website
   ```

2. **Update Strategy**: You can fine-tune the update strategy in your deployment YAML:

   ```yaml
   spec:
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 25%
         maxSurge: 25%
   ```

   This ensures that no more than 25% of the pods are unavailable during the update, and no more than 25% extra pods are created.

By following these steps, you've successfully performed a rolling update of your e-commerce application, adding a new promotional banner for your marketing campaign. This demonstrates how Kubernetes can manage updates with zero downtime, maintaining service availability while improving your application.
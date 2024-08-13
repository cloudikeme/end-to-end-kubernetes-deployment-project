# Step 10: Autoscale My Application

Certainly! I'll guide you through the process of implementing Horizontal Pod Autoscaler (HPA) for your e-commerce application. This will allow your application to automatically scale based on CPU usage, helping it handle unpredictable traffic spikes efficiently.

### **Implement HPA**

First, we'll create an HPA targeting 50% CPU utilization, with a minimum of 2 and a maximum of 10 pods. We can do this either by creating a YAML file or using a kubectl command.

Let's create a YAML file named `ecommerce-hpa.yaml`:

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-website
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
```

### **Apply HPA**

Now, let's apply this HPA configuration:

```bash
kubectl apply -f ecommerce-hpa.yaml
```

Alternatively, you can use the kubectl command as you suggested:

```bash
kubectl autoscale deployment ecommerce-website --cpu-percent=50 --min=2 --max=10
```

1. **Verify HPA**

Check that the HPA has been created:

```bash
kubectl get hpa
```

You should see output similar to:

```
NAME            REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
ecommerce-hpa   Deployment/ecommerce-website  0%/50%    2         10        2          30s
```

### **Simulate Load**

To simulate load, we'll use Apache Bench (ab). If you don't have it installed, you can install it on most Unix-based systems with:

```bash
sudo apt-get install apache2-utils  # For Ubuntu/Debian
# or
brew install ab  # For macOS with Homebrew
```

First, get the external IP of your service:

```bash
kubectl get service ecommerce-website-service
```

Now, let's generate some load. Replace `<EXTERNAL-IP>` with your actual external IP:

```bash
ab -n 100000 -c 100 http://<EXTERNAL-IP>/
```

This command sends 100,000 requests with 100 concurrent connections.

### **Monitor Autoscaling**

While the load is being generated, monitor the HPA:

```bash
kubectl get hpa -w
```

You should see the CPU usage increase and the number of replicas go up:

```
NAME            REFERENCE                     TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
ecommerce-hpa   Deployment/ecommerce-website  0%/50%     2         10        2          1m
ecommerce-hpa   Deployment/ecommerce-website  60%/50%    2         10        2          2m
ecommerce-hpa   Deployment/ecommerce-website  80%/50%    2         10        3          3m
ecommerce-hpa   Deployment/ecommerce-website  65%/50%    2         10        4          4m
ecommerce-hpa   Deployment/ecommerce-website  55%/50%    2         10        4          5m
```

You can also watch the pods being created:

```bash
kubectl get pods -w
```

1. **Verify Scaling Down**

After the load decreases, the HPA will gradually scale down the number of pods. This may take a few minutes due to the default cooldown period.

### **Outcome**:
The deployment automatically adjusts the number of pods based on CPU load, showcasing Kubernetes' capability to maintain performance under varying loads.

**Additional Considerations**:

In production, i would consider the following:

1. **Resource Requests**: Ensure your deployment specifies CPU requests, as HPA uses these to calculate the target CPU utilization:

   ```yaml
   resources:
     requests:
       cpu: 100m
   ```

2. **Metrics Server**: HPA requires the Metrics Server to be running in your cluster. Most managed Kubernetes services have this pre-installed, but you may need to install it manually on some setups.

3. **Custom Metrics**: You can also set up autoscaling based on custom metrics, such as requests per second or queue length, which might be more relevant for your specific application.

4. **Vertical Pod Autoscaler**: Consider using Vertical Pod Autoscaler (VPA) in conjunction with HPA to optimize both the number of pods and their resource allocations.

5. **Cluster Autoscaler**: If you're running on a cloud provider, consider setting up Cluster Autoscaler to automatically adjust the number of nodes in your cluster based on resource demands.

6. **Cooldown/Delay**: Be aware that there's a cooldown period (default is 5 minutes) before scaling down to prevent flapping. You can adjust this if needed.

7. **Monitoring**: Set up comprehensive monitoring and alerting to keep track of your application's performance and scaling behavior.

By implementing Horizontal Pod Autoscaler, you've significantly improved your e-commerce application's ability to handle varying loads. This automated scaling ensures that your application remains responsive during traffic spikes while efficiently using resources during periods of low demand.
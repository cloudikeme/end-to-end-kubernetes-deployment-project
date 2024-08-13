# Step 7: Scale Your Application

In this step you would scale your e-commerce application to prepare for the expected increase in traffic due to the marketing campaign. This will demonstrate Kubernetes' ability to dynamically manage application scalability.

### **Evaluate Current Load**

First, let's check the current number of running pods:

```bash
kubectl get pods
```

You should see output similar to this:

```
NAME                                 READY   STATUS    RESTARTS   AGE
ecommerce-website-5f7b9d4f9d-abcd1   1/1     Running   0          2h
ecommerce-website-5f7b9d4f9d-efgh2   1/1     Running   0          2h
```

This shows that you currently have 2 replicas running, as specified in your original deployment.

### **Scale Up**

To handle the tripled traffic, let's increase the number of replicas to 6. You can do this in two ways:

a. By editing the deployment YAML file:

Edit your `website-deployment.yaml` file and change the `replicas` field:

```yaml
spec:
  replicas: 6
```

Then apply the changes:

```bash
kubectl apply -f website-deployment.yaml
```

b. Or, use the `kubectl scale` command:

```bash
kubectl scale deployment/ecommerce-website --replicas=6
```

### **Monitor Scaling**

Now, let's observe the deployment scaling up:

```bash
kubectl get pods -w
```

The `-w` flag will watch for changes. You should see new pods being created:

```
NAME                                 READY   STATUS              RESTARTS   AGE
ecommerce-website-5f7b9d4f9d-abcd1   1/1     Running             0          2h
ecommerce-website-5f7b9d4f9d-efgh2   1/1     Running             0          2h
ecommerce-website-5f7b9d4f9d-ijkl3   0/1     ContainerCreating   0          2s
ecommerce-website-5f7b9d4f9d-mnop4   0/1     ContainerCreating   0          2s
ecommerce-website-5f7b9d4f9d-qrst5   0/1     ContainerCreating   0          2s
ecommerce-website-5f7b9d4f9d-uvwx6   0/1     ContainerCreating   0          2s
```

Wait until all pods are in the Running state.

1. **Verify the Scaling**

To confirm that the scaling operation was successful, run:

```bash
kubectl get deployment ecommerce-website
```

You should see output similar to:

```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
ecommerce-website    6/6     6            6           2h
```

This indicates that all 6 replicas are now ready and available.

5. **Check the Service**

Ensure that your LoadBalancer service is correctly routing traffic to all pods:

```bash
kubectl describe service ecommerce-website-service
```

Look for the "Endpoints" section, which should list the IP addresses of all 6 pods.

### Outcome**
Your application has now scaled up to handle the increased load. Kubernetes has dynamically created new pods to match the desired state of 6 replicas. This demonstrates Kubernetes' ability to easily manage application scalability.

**Additional Considerations**:

1. **Monitor Application Performance**: Use Kubernetes' built-in monitoring tools or integrate with external monitoring solutions to keep an eye on your application's performance under increased load.

2. **Autoscaling**: For long-term management, consider setting up Horizontal Pod Autoscaler (HPA) to automatically adjust the number of replicas based on CPU utilization or custom metrics.

3. **Database Scaling**: Ensure that your database can handle the increased load. You might need to scale your database or implement caching mechanisms.

4. **Load Testing**: Before the actual marketing campaign, perform load testing to ensure your scaled application can handle the expected traffic increase.

5. **Resource Limits**: Make sure you have enough resources in your Kubernetes cluster to handle the increased number of pods. You may need to scale your cluster if you're approaching resource limits.

By following these steps, you've successfully scaled your e-commerce application to handle the expected increase in traffic from your marketing campaign. This showcases how Kubernetes can help manage application scalability dynamically, allowing you to respond quickly to changing demand.
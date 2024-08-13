# Step 11: Implement Liveness and Readiness Probes

In this step, i will be implementing liveness and readiness probes for my e-commerce web application. These probes help Kubernetes ensure that your application is running correctly and is ready to receive traffic.

What are Probes?

### **Define Probes**

First, let's modify your `website-deployment.yaml` to include liveness and readiness probes. We'll assume your application has a `/health` endpoint for the liveness probe and a `/ready` endpoint for the readiness probe.

Update your `website-deployment.yaml`:

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
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
```

In this configuration:
- The liveness probe checks the `/health` endpoint every 10 seconds, starting 15 seconds after the container starts.
- The readiness probe checks the `/ready` endpoint every 10 seconds, starting 5 seconds after the container starts.
- If the liveness probe fails 3 times consecutively, Kubernetes will restart the container.
- If the readiness probe fails, Kubernetes will stop sending traffic to the pod until it passes.

### **Apply Changes**

Apply the updated deployment configuration:

```bash
kubectl apply -f website-deployment.yaml
```

### **Test Probes**

To test the probes, we'll simulate different scenarios:

a. Simulate an unresponsive application (failing liveness probe):

First, exec into one of the pods:

```bash
kubectl exec -it $(kubectl get pods -l app=ecommerce-website -o jsonpath="{.items[0].metadata.name}") -- /bin/bash
```

Then, simulate an application crash by killing the main process:

```bash
kill 1
```

Exit the pod and observe what happens:

```bash
kubectl get pods -w
```

You should see Kubernetes automatically restart the pod.

b. Simulate an application that's not ready (failing readiness probe):

For this, you'd need to have implemented a way to toggle the readiness of your application. For example, you could have an endpoint that toggles a "maintenance mode":

```bash
curl -X POST http://<EXTERNAL-IP>/toggle-maintenance
```

Then observe the pod status:

```bash
kubectl get pods -o wide
```

You should see that the pod is running but not ready.

### **Verify Probe Functionality**

You can check the events related to your pods to see probe-related actions:

```bash
kubectl describe pods
```

Look for events related to liveness and readiness probes in the output.

### **Outcome**

With these probes in place:
- If my application becomes unresponsive, Kubernetes will automatically restart the pod.
- If my application is not ready to serve traffic (e.g., still initializing), Kubernetes will not send traffic to it until it's ready.

This enhances my application's reliability and availability.

**Additional Considerations**:

1. **Probe Endpoints**: Ensure my `/health` and `/ready` endpoints are lightweight and don't consume too many resources.

2. **Timing**: Adjust the `initialDelaySeconds`, `periodSeconds`, and `failureThreshold` based on your application's specific characteristics.

3. **Graceful Shutdown**: Implement graceful shutdown in your application to handle SIGTERM signals when Kubernetes needs to stop a pod.

4. **Startup Probe**: For applications with longer startup times, consider using a startup probe in addition to liveness and readiness probes.

5. **Monitoring**: Set up monitoring for probe failures to get insights into your application's health over time.

6. **Different Probe Types**: While we used HTTP probes here, Kubernetes also supports TCP and command probes. Choose the type that best fits your application.

Here's a summary of the key commands used:

```bash
# Apply the updated deployment
kubectl apply -f website-deployment.yaml

# Exec into a pod
kubectl exec -it $(kubectl get pods -l app=ecommerce-website -o jsonpath="{.items[0].metadata.name}") -- /bin/bash

# Watch pod status
kubectl get pods -w

# Describe pods to see probe-related events
kubectl describe pods
```

By implementing these liveness and readiness probes, i've significantly improved the reliability and availability of my e-commerce application. Kubernetes can now automatically detect and respond to various failure scenarios, ensuring that my application remains healthy and responsive.
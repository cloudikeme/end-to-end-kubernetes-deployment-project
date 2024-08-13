# Step 9: Roll Back a Deployment

In this step you would be rolling back your deployment to the previous version after discovering a bug in the new promotional banner. This scenario demonstrates the importance of having a quick and reliable rollback strategy in place.

### **Identify Issue**

Let's assume your observability/monitoring tools have alerted you to a problem affecting user experience after deploying the new version with the promotional banner. This could be increased error rates, slower response times, or other issues.

### **Roll Back**

To revert to the previous deployment state, execute the following command:

```bash
kubectl rollout undo deployment/ecommerce-website
```

This command will initiate a rollback to the previous known stable version of your deployment.

### **Monitor Rollback**

You can watch the rollback process in real-time:

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

### **Verify Rollback**

To confirm that all pods are now running the previous version:

```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
```

You should see all pods using the `:v1` image (or whatever tag you used for the previous version).

### **Check the Website**

Access your website through the LoadBalancer service. Verify that the promotional banner is no longer present and the site has returned to its pre-update state.

**Review Deployment History:**

You can review the deployment history to understand what versions have been deployed:

```bash
kubectl rollout history deployment/ecommerce-website
```

This will show you a list of revisions, including the one you just rolled back from.

### **Outcome**

The application's stability is quickly restored by rolling back to the previous known-good version. This highlights the importance of having a rollback strategy in your deployment process.

**Additional Considerations**:

In a typical production env, i would do the following: 

1. **Root Cause Analysis**: After stabilizing the application, investigate the cause of the bug in the new version. This might involve code review, testing, and potentially reproducing the issue in a non-production environment.

2. **Update Your Code**: Fix the bug in your codebase and create a new version that includes both the fix and the promotional banner.

3. **Versioning**: Consider using more specific version tags for your Docker images (e.g., semantic versioning) to make it easier to track which version introduced issues.

4. **Automated Testing**: Implement or improve automated testing in your CI/CD pipeline to catch potential bugs before they reach production.

5. **Gradual Rollout**: For future updates, consider using strategies like canary deployments or blue-green deployments to catch issues before they affect all users.

6. **Monitoring and Alerting**: Ensure your monitoring tools are set up to quickly detect and alert on any anomalies after a deployment.

7. **Documentation**: Document the issue and the rollback process for future reference and to improve your deployment procedures.

Here's a summary of the commands used in this rollback process:

```bash
# Rollback the deployment
kubectl rollout undo deployment/ecommerce-website

# Monitor the rollback status
kubectl rollout status deployment/ecommerce-website

# Verify the image version of the pods
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image

# Review deployment history
kubectl rollout history deployment/ecommerce-website
```

By following these steps, i have successfully rolled back my e-commerce application to a stable version after encountering a bug. This demonstrates the flexibility and resilience that Kubernetes provides in managing application deployments and quickly responding to issues.
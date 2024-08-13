# Step 13: Package Everything in Helm

Certainly! I'll guide you through the process of packaging your e-commerce application using Helm. This will make your deployment and management on Kubernetes clusters more efficient and scalable.

### **Create Helm Chart**

First, let's create a new Helm chart for your e-commerce application:

```bash
helm create ecommerce-app
```

This command creates a new directory named `ecommerce-app` with the default Helm chart structure. We'll modify these files to fit your application.

### **Define Chart Structure**

Navigate to the `ecommerce-app` directory and update the files as follows:

#### Update `Chart.yaml`:

```yaml
apiVersion: v2
name: ecommerce-app
description: A Helm chart for the e-commerce application
version: 0.1.0
appVersion: "1.0.0"
```

#### Modify `values.yaml`:

```yaml
# Default values for ecommerce-app.
replicaCount: 3

image:
  repository: yourdockerhubusername/ecom-web
  tag: "v2"
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 80

ingress:
  enabled: false

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50

database:
  host: mysql-service
  name: ecommerce_db

featureToggles:
  darkMode: true
  newProductPage: false
```

#### Create templates for your Kubernetes resources in the `templates` directory:

- `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "ecommerce-app.fullname" . }}
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "ecommerce-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "ecommerce-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
                  key: DB_HOST
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
                  key: DB_NAME
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
                  key: DB_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
                  key: DB_PASSWORD
            - name: FEATURE_DARK_MODE
              value: "{{ .Values.featureToggles.darkMode }}"
            - name: FEATURE_NEW_PRODUCT_PAGE
              value: "{{ .Values.featureToggles.newProductPage }}"
          livenessProbe:
            httpGet:
              path: /health
              port: http
          readinessProbe:
            httpGet:
              path: /ready
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

- `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "ecommerce-app.fullname" . }}
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "ecommerce-app.selectorLabels" . | nindent 4 }}
```

- `hpa.yaml`:

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "ecommerce-app.fullname" . }}
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "ecommerce-app.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
{{- end }}
```

- `secrets.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
type: Opaque
stringData:
  DB_HOST: {{ .Values.database.host | quote }}
  DB_NAME: {{ .Values.database.name | quote }}
  DB_USER: {{ .Values.database.user | quote }}
  DB_PASSWORD: {{ .Values.database.password | quote }}
```

### **Package and Deploy**

Now that we have our Helm chart set up, let's package and deploy it:

#### Package the chart:

```bash
helm package ./ecommerce-app
```

This will create a `.tgz` file in your current directory.

#### Install the chart:

```bash
helm install my-ecommerce-app ./ecommerce-app-0.1.0.tgz
```

### **Verify the Deployment**

Check that all resources have been created:

```bash
kubectl get all,secrets,configmaps -l app.kubernetes.io/instance=my-ecommerce-app
```

### **Customize the Deployment**

You can customize the deployment by creating a `custom-values.yaml` file:

```yaml
replicaCount: 4
image:
  tag: "v3"
featureToggles:
  darkMode: false
```

And then upgrade the release:

```bash
helm upgrade my-ecommerce-app ./ecommerce-app-0.1.0.tgz -f custom-values.yaml
```

### **Outcome**:

My e-commerce application is now packaged as a Helm chart, simplifying deployment processes and enabling easy versioning and rollback capabilities.

**Additional Considerations**:

1. **Version Control**: Store your Helm chart in version control alongside your application code.

2. **Chart Repository**: Consider setting up a Helm chart repository to store and distribute your charts.

3. **Dependencies**: If your application depends on other services (like a database), you can define these as dependencies in your Helm chart.

4. **Testing**: Use `helm lint` and `helm template` to test your chart before deployment.

5. **Namespaces**: Consider using namespaces in your Helm chart for better isolation.

6. **Helm Hooks**: Utilize Helm hooks for tasks like database migrations or post-install setup.

7. **Documentation**: Provide clear documentation in your chart's README about its usage and configuration options.

Here's a summary of the key Helm commands i used:

```bash
# Create a new chart
helm create ecommerce-app

# Package the chart
helm package ./ecommerce-app

# Install the chart
helm install my-ecommerce-app ./ecommerce-app-0.1.0.tgz

# Upgrade the release
helm upgrade my-ecommerce-app ./ecommerce-app-0.1.0.tgz -f custom-values.yaml

# List releases
helm list

# Uninstall a release
helm uninstall my-ecommerce-app
```

By packaging my application with Helm, i've made it much easier to deploy, upgrade, and manage my e-commerce application across different Kubernetes environments. This approach provides consistency and reproducibility in my deployments, which is crucial for maintaining a reliable application as it scales.
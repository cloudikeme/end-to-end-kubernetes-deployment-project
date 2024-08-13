I'll be implementing persistent storage for my MariaDB database. This will ensure that my data persists across pod restarts and redeployments.

Let's update our Helm chart to include persistent storage for MariaDB.

### **Create a PersistentVolumeClaim (PVC)**

First, let's add a PVC definition to our Helm chart. Create a new file called `pvc.yaml` in the `templates` directory of your Helm chart:

```yaml
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "ecommerce-app.fullname" . }}-mariadb-data
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
spec:
  accessModes:
    - {{ .Values.persistence.accessMode | quote }}
  resources:
    requests:
      storage: {{ .Values.persistence.size | quote }}
  {{- if .Values.persistence.storageClass }}
  {{- if (eq "-" .Values.persistence.storageClass) }}
  storageClassName: ""
  {{- else }}
  storageClassName: "{{ .Values.persistence.storageClass }}"
  {{- end }}
  {{- end }}
{{- end }}
```

### **Update MariaDB Deployment**

Now, let's modify the MariaDB deployment to use this PVC. Update the `mariadb-deployment.yaml` file in your `templates` directory:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "ecommerce-app.fullname" . }}-mariadb
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
    app.kubernetes.io/component: database
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "ecommerce-app.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: database
  template:
    metadata:
      labels:
        {{- include "ecommerce-app.selectorLabels" . | nindent 8 }}
        app.kubernetes.io/component: database
    spec:
      containers:
        - name: mariadb
          image: "{{ .Values.mariadb.image.repository }}:{{ .Values.mariadb.image.tag }}"
          imagePullPolicy: {{ .Values.mariadb.image.pullPolicy }}
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
                  key: DB_PASSWORD
            - name: MYSQL_DATABASE
              value: {{ .Values.database.name | quote }}
          ports:
            - name: mysql
              containerPort: 3306
              protocol: TCP
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
      volumes:
        - name: data
          {{- if .Values.persistence.enabled }}
          persistentVolumeClaim:
            claimName: {{ include "ecommerce-app.fullname" . }}-mariadb-data
          {{- else }}
          emptyDir: {}
          {{- end }}
```

### **Update values.yaml**

Add the following to your `values.yaml` file to configure the persistent storage:

```yaml
mariadb:
  image:
    repository: mariadb
    tag: "10.5"
    pullPolicy: IfNotPresent

persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 1Gi
  # storageClass: "-"

database:
  name: ecommerce_db
  user: ecommerce_user
  # password should be set via --set database.password=<password> during helm install/upgrade
```

### **Update secrets.yaml**

Make sure your `secrets.yaml` file includes the MariaDB credentials:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "ecommerce-app.fullname" . }}-db-credentials
type: Opaque
stringData:
  DB_HOST: {{ include "ecommerce-app.fullname" . }}-mariadb
  DB_NAME: {{ .Values.database.name | quote }}
  DB_USER: {{ .Values.database.user | quote }}
  DB_PASSWORD: {{ .Values.database.password | quote }}
```

### **Create a MariaDB Service**

Create a `mariadb-service.yaml` file in the `templates` directory:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "ecommerce-app.fullname" . }}-mariadb
  labels:
    {{- include "ecommerce-app.labels" . | nindent 4 }}
    app.kubernetes.io/component: database
spec:
  ports:
    - port: 3306
      targetPort: mysql
      protocol: TCP
      name: mysql
  selector:
    {{- include "ecommerce-app.selectorLabels" . | nindent 4 }}
    app.kubernetes.io/component: database
```

### **Deploy the Updated Chart**

Now you can deploy or upgrade your Helm chart with the new persistent storage configuration:

```bash
helm upgrade --install my-ecommerce-app ./ecommerce-app \
  --set database.password=<your-secure-password> \
  --set persistence.enabled=true \
  --set persistence.size=1Gi
```

### **Outcome**

With these changes, my MariaDB database will now use persistent storage. The database data will be stored in a PersistentVolume, which will persist even if the MariaDB pod is deleted or rescheduled. This ensures data durability across pod restarts and redeployments.

**Additional Considerations**:

1. **Backups**: Even with persistent storage, regular backups are crucial. I will Consider implementing a backup solution.

2. **Storage Class**: Depending on my Kubernetes environment, i might need to specify a particular storage class. I know i can do this by setting `persistence.storageClass` in my values or during the Helm install/upgrade.

3. **Scaling**: Note that this setup uses a single MariaDB instance. For production environments, you might want to consider a more robust solution like a MariaDB Galera cluster for high availability.

4. **Resource Limits**: Consider adding resource requests and limits for your MariaDB container to ensure it has the resources it needs and doesn't consume more than it should.

5. **Initialization**: You might want to add an init container or a post-install hook to initialize the database schema if it doesn't exist.

6. **Monitoring**: Set up monitoring for your database to keep track of its performance and usage.

Here's a summary of the key commands:

```bash
# Install/upgrade the Helm chart with persistent storage
helm upgrade --install my-ecommerce-app ./ecommerce-app \
  --set database.password=<your-secure-password> \
  --set persistence.enabled=true \
  --set persistence.size=1Gi

# Check the status of your PVC
kubectl get pvc

# Check the status of your MariaDB pod
kubectl get pods -l app.kubernetes.io/component=database
```

By implementing persistent storage, i've significantly improved the reliability and durability of your e-commerce application's data layer. This is a critical step in preparing your application for production use, ensuring that your valuable data is preserved across various operational scenarios.
# Implement Configuration Management

### **Modify the Web Application**

First, let's assume you've modified your PHP application to check for an environment variable `FEATURE_DARK_MODE`. Here's a simple example of how this might look in your PHP code:

We'll need to modify the HTML, add some CSS, and include a bit of JavaScript to implement this feature. Here's how we can do it:

1. First, let's modify the `header.php` file to include a toggle switch:

```php
<!-- views/header.php -->
<!-- ... (previous code) ... -->
<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
    <ul class="nav navbar-nav main_nav">
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/contact">Contact us</a></li>
    </ul>
    <ul class="nav navbar-nav navbar-right">
        <li><a href="#" class="nav_searchFrom"><i class="lnr lnr-magnifier"></i></a></li>
        <li>
            <div class="dark-mode-toggle">
                <input type="checkbox" id="darkModeToggle" class="dark-mode-input">
                <label for="darkModeToggle" class="dark-mode-label">Dark Mode</label>
            </div>
        </li>
    </ul>
</div>
<!-- ... (rest of the code) ... -->
```


2. Now, let's add some CSS for the dark mode. Create a new file `css/dark-mode.css`:

```css
/* css/dark-mode.css */
body.dark-mode {
    background-color: #1a1a1a;
    color: #ffffff;
}

.dark-mode .navbar {
    background-color: #2c2c2c;
}

.dark-mode .navbar-default .navbar-nav > li > a {
    color: #ffffff;
}

.dark-mode .business_content {
    background-color: #2c2c2c;
}

.dark-mode .footer_area {
    background-color: #2c2c2c;
}

.dark-mode-toggle {
    display: flex;
    align-items: center;
}

.dark-mode-input {
    display: none;
}

.dark-mode-label {
    cursor: pointer;
    padding: 5px 10px;
    border-radius: 15px;
    background-color: #ccc;
    color: #333;
    transition: background-color 0.3s, color 0.3s;
}

.dark-mode-input:checked + .dark-mode-label {
    background-color: #333;
    color: #fff;
}
```

3. Include the new CSS file in the `header.php`:

```php
<!-- views/header.php -->
<!-- ... (previous code) ... -->
<link href="css/style.css" rel="stylesheet">
<link href="css/dark-mode.css" rel="stylesheet">
<!-- ... (rest of the code) ... -->
```

4. Add JavaScript to handle the dark mode toggle. Create a new file `js/dark-mode.js`:

```javascript
// js/dark-mode.js
document.addEventListener('DOMContentLoaded', (event) => {
    const darkModeToggle = document.getElementById('darkModeToggle');
    const body = document.body;

    // Check if dark mode preference is saved in localStorage
    if (localStorage.getItem('darkMode') === 'enabled') {
        body.classList.add('dark-mode');
        darkModeToggle.checked = true;
    }

    darkModeToggle.addEventListener('change', () => {
        if (darkModeToggle.checked) {
            body.classList.add('dark-mode');
            localStorage.setItem('darkMode', 'enabled');
        } else {
            body.classList.remove('dark-mode');
            localStorage.setItem('darkMode', null);
        }
    });
});
```

5. Include the new JavaScript file in the `footer.php`:

```php
<!-- views/footer.php -->
<!-- ... (previous code) ... -->
<script src="js/theme.js"></script>
<script src="js/dark-mode.js"></script>
</body>
</html>
```

### DARKMODE CONFIG

**1. First, let's create a ConfigMap YAML file. We'll call it `darkmode-config.yaml`:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: darkmode-config
data:
  DARK_MODE_ENABLED: "true"
```

**2. Apply this ConfigMap to your Kubernetes cluster:**

```bash
kubectl apply -f darkmode-config.yaml
```

**3. Modify your application's Deployment YAML to use this ConfigMap. Add an environment variable that reads from the ConfigMap:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudikeme-store
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cloudikeme-store
  template:
    metadata:
      labels:
        app: cloudikeme-store
    spec:
      containers:
      - name: cloudikeme-store
        image: your-image:tag
        env:
        - name: DARK_MODE_ENABLED
          valueFrom:
            configMapKeyRef:
              name: darkmode-config
              key: DARK_MODE_ENABLED
        # ... other configuration ...
```

4. Now, let's modify our PHP code to read this environment variable. Update the `header.php` file:

```php
<!-- views/header.php -->
<?php
$darkModeEnabled = getenv('DARK_MODE_ENABLED') === 'true';
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- ... other head elements ... -->
    <?php if ($darkModeEnabled): ?>
    <link href="css/dark-mode.css" rel="stylesheet">
    <?php endif; ?>
</head>
<body<?php echo $darkModeEnabled ? ' class="dark-mode"' : ''; ?>>
    <!-- ... rest of the body ... -->
    <?php if ($darkModeEnabled): ?>
    <ul class="nav navbar-nav navbar-right">
        <li><a href="#" class="nav_searchFrom"><i class="lnr lnr-magnifier"></i></a></li>
        <li>
            <div class="dark-mode-toggle">
                <input type="checkbox" id="darkModeToggle" class="dark-mode-input">
                <label for="darkModeToggle" class="dark-mode-label">Dark Mode</label>
            </div>
        </li>
    </ul>
    <?php endif; ?>
    <!-- ... rest of the header ... -->
```

5. Update the `footer.php` to conditionally include the dark mode JavaScript:

```php
<!-- views/footer.php -->
    <!-- ... other scripts ... -->
    <?php if ($darkModeEnabled): ?>
    <script src="js/dark-mode.js"></script>
    <?php endif; ?>
</body>
</html>
```

6. Modify the `dark-mode.js` file to check the initial state from the body class:

```javascript
// js/dark-mode.js
document.addEventListener('DOMContentLoaded', (event) => {
    const darkModeToggle = document.getElementById('darkModeToggle');
    const body = document.body;

    // Check if dark mode is initially enabled
    if (body.classList.contains('dark-mode')) {
        darkModeToggle.checked = true;
    }

    darkModeToggle.addEventListener('change', () => {
        if (darkModeToggle.checked) {
            body.classList.add('dark-mode');
        } else {
            body.classList.remove('dark-mode');
        }
    });
});
```






















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


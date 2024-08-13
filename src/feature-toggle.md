Certainly! I'll add a feature toggle for a "dark mode" to the web application. We'll need to modify the HTML, add some CSS, and include a bit of JavaScript to implement this feature. Here's how we can do it:

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

These changes will add a dark mode toggle to your web application. The dark mode state will be saved in the browser's localStorage, so it will persist across page reloads and visits.

To summarize the changes:

1. We added a toggle switch in the navigation bar.
2. We created a new CSS file for dark mode styles.
3. We added JavaScript to handle the toggle functionality and save the user's preference.

You may need to adjust the CSS further to ensure all elements of your site look good in dark mode. Also, you might want to add more specific styles for different sections of your website to enhance the dark mode experience.




Certainly! We can use a Kubernetes ConfigMap to manage the feature toggle for the dark mode. This approach allows us to enable or disable the feature without changing the application code. Here's how we can implement this:

1. First, let's create a ConfigMap YAML file. We'll call it `darkmode-config.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: darkmode-config
data:
  DARK_MODE_ENABLED: "true"
```

2. Apply this ConfigMap to your Kubernetes cluster:

```bash
kubectl apply -f darkmode-config.yaml
```

3. Modify your application's Deployment YAML to use this ConfigMap. Add an environment variable that reads from the ConfigMap:

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

With these changes, the dark mode feature can be toggled on or off by updating the ConfigMap in Kubernetes. When `DARK_MODE_ENABLED` is set to "true" in the ConfigMap, the dark mode CSS will be included, the body will have the "dark-mode" class by default, and the toggle switch will be visible.

To disable the feature, you would update the ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: darkmode-config
data:
  DARK_MODE_ENABLED: "false"
```

Then apply the updated ConfigMap:

```bash
kubectl apply -f darkmode-config.yaml
```

And finally, restart your application pods to pick up the new configuration:

```bash
kubectl rollout restart deployment cloudikeme-store
```

This approach allows you to enable or disable the dark mode feature without changing your application code, making it easy to manage in a Kubernetes environment.
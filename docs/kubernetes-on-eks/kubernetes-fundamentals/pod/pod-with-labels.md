---
description: Discover the use of labels with kubernetes pods in our guide. Learn how labels enhance organization and management in containerized applications.
---

# Kubernetes Pod With Labels

Labels are key/value pairs that are attached to Kubernetes objects, such as pods.

Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}


## Step 1: Create Pods With Labels

=== ":octicons-file-code-16: `frontend.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-frontend-pod
      labels:
        app: nginx-frontend
        tier: frontend
        environment: dev
    spec:
      containers:
      - name: nginx
        image: reyanshkharga/nginx:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 80
    ```

=== ":octicons-file-code-16: `backend.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-backend-pod
      labels:
        app: nginx-backend
        tier: backend
        environment: prod
    spec:
      containers:
      - name: nginx
        image: reyanshkharga/nginx:v2
        imagePullPolicy: Always
        ports:
        - containerPort: 80
    ```

Apply the manifest files to create pods:

```
# Create frontend pod
kubectl apply -f frontend.yml

# Create backend pod
kubectl apply -f backend.yml
```

## Step 2: List Pods

```
# List pods
kubectl get pods

# List pods and show labels
kubectl get pods --show-labels
```


## Step 3: Filter Pods Using Labels

1. Equality based filtering:

    ```
    kubectl get pods -l environment=prod
    kubectl get pods -l environment=dev
    kubectl get pods -l environment=qa

    kubectl get pods -l environment=prod,tier=backend
    kubectl get pods -l environment=prod,tier=frontend
    kubectl get pods -l environment=dev,tier=frontend
    kubectl get pods -l environment=qa,tier=backend
    ```

2. Set based filtering:

    ```
    kubectl get pods -l 'environment in (prod, qa)'
    kubectl get pods -l 'environment in (dev, qa)'
    kubectl get pods -l 'environment in (prod, dev)'
    kubectl get pods -l 'environment in (prod, dev, qa)'

    kubectl get pods -l 'environment in (prod, qa),tier in (backend, frontend)'
    kubectl get pods -l 'environment in (dev, qa),tier in (backend, frontend)'
    kubectl get pods -l 'environment in (prod, dev),tier in (backend, frontend)'
    kubectl get pods -l 'environment in (prod, dev),tier in (database)'
    ```

## Clean Up

```
# Delete frontend pod
kubectl delete -f frontend.yml

# Delete backend pod
kubectl delete -f backend.yml
```


<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx
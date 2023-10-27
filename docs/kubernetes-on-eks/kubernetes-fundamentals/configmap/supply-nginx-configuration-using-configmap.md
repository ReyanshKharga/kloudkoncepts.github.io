---
description: Simplify Nginx configuration by supplying files through ConfigMap in Kubernetes. Discover how to streamline your web server setup with our step-by-step guide. Enhance your Nginx configuration management now!
---

# Supply Nginx Configuration Using ConfigMap

Now, let's see how you can use `ConfigMap` to supply configuration to applications. In this example, we will use `ConfigMap` to provide configuration to the nginx application.


## Nginx Deployment Without ConfigMap

First, let's explore the default behaviour of `nginx` application.

By default, Nginx serves responses from a virtual server that listens on port `80` and serves the default `index.html` file located at `/usr/share/nginx/html`.

Let's test this out!


### 1. Create nginx deployment manifest

=== ":octicons-file-code-16: `nginx-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx
    ```

### 2. Apply the manifest to create the nginx deployment

```
kubectl apply -f nginx-deployment.yml
```

### 3. Verify deployment and pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```

### 4. View default nginx configuration files

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# View the content of the default.conf file
cat /etc/nginx/conf.d/default.conf

# View the content of the default web pages
cd /usr/share/nginx/html
ls
cat index.html
```

Observe the following:

- The nginx server listens on port `80` by default
- Nginx serves the default `index.html` page at `/usr/share/nginx/html`

Get response from nginx server:

```
# Hit port 80
curl localhost
{OR}
curl localhost:80
```

What if you want nginx to serve response from port `8080` instead of port `80`?

`ConfigMap` can help. Let's try it out!



## Step 1: Create a ConfigMap That Stores Nginx Configuration

First, let's create a `ConfigMap` that stores nginx configuration as follows:


=== ":octicons-file-code-16: `nginx-configmap.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nginx-configmap
    data:
      default.conf: |
        server {
          listen       8080;
          listen  [::]:8080;
          server_name  localhost;

          #access_log  /var/log/nginx/host.access.log  main;

          location / {
              root   /usr/share/nginx/html;
              index  index.html index.htm;
          }

          #error_page  404              /404.html;

          # redirect server error pages to the static page /50x.html
          #
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
        }
    ```

Observe the following:

- The nginx server listens on port `8080`
- Nginx will serve the default `index.html` page at `/usr/share/nginx/html`

Apply the manifest to create ConfigMap:

```
kubectl apply -f nginx-configmap.yml
```


## Step 2: Verify ConfigMap

```
# List configmaps
kubectl get cm

# Describe the configmap
kubectl describe cm nginx-configmap
```


## Step 3: Update Nginx Deployment to Use Configuration From ConfigMap

Let's update the nginx deployment to use nginx configuration from the `ConfigMap` we created in the previous step.

The updated deployment should look like the following:

=== ":octicons-file-code-16: `nginx-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx
            volumeMounts:
            - name: my-volume
              mountPath: "/etc/nginx/conf.d"
          volumes:
          - name: my-volume
            configMap:
              name: nginx-configmap
    ```


Apply the manifest to update the deployment:

```
kubectl apply -f nginx-deployment.yml
```


## Step 4: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods

# Describe the Pod
kubectl describe pod <pod-name>
```


## Step 5: View Nginx Configuration Files

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# View the content of the default.conf file
cat /etc/nginx/conf.d/default.conf

# View the content of the default web pages
cd /usr/share/nginx/html
ls
cat index.html
```

Observe the following:

- The nginx server now listens on port `8080`
- Nginx serves the default `index.html` page at `/usr/share/nginx/html`

Get response from nginx server:

```
# Hit port 8080
curl localhost:8080
```


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- nginx-deployment.yml
│   |-- nginx-configmap.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```
---
description: Simplify your Nginx configuration management in Kubernetes by supplying multiple configurations using ConfigMap. Learn how to efficiently organize and optimize your web server settings with our comprehensive guide. 
---

# Supply Multiple Nginx Configuration Using ConfigMap

In the previous lesson, we learned how to use a `ConfigMap` to supply nginx configuration file to nginx. More specifically, we used a `ConfigMap` to modify the default nginx port.

In this example, we will explore how to use a `ConfigMap` to supply multiple nginx configuration files. Specifically, we will learn how to use a `ConfigMap` to modify the default nginx port and serve a custom HTML page.



## Step 1: Create a ConfigMap That Stores Nginx Configuration

Let's create a `ConfigMap` that stores nginx configuration:

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
      index.html: |
        <!DOCTYPE html>
          <html>
            <head>
              <title>Welcome to My Website!</title>
            </head>
            <body>
              <h1>Welcome to My Website!</h1>
            </body>
        </html>
    ```

Observe the following:

- The nginx server listens on port `8080`
- Nginx will serve the modified `index.html` page at `/usr/share/nginx/html`

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


## Step 3: Create Nginx Deployment That Uses Configuration from ConfigMap

Let's create nginx deployment that uses configuration files from the `ConfigMap` we created in the previous step:

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
              mountPath: "/etc/nginx/conf.d/default.conf"
              subPath: "default.conf"
            - name: my-volume
              mountPath: "/usr/share/nginx/html/index.html"
              subPath: "index.html"
          volumes:
          - name: my-volume
            configMap:
              name: nginx-configmap
    ```

Observe the following:

- We have created a volume from `nginx-configmap` ConfigMap
- We have mounted `default.conf` file from `nginx-configmap` to the `nginx` container at `/etc/nginx/conf.d/default.conf`
- We have mounted `index.html` file from `nginx-configmap` to the `nginx` container at `/usr/share/nginx/html/index.html`

!!! note
    By using `subPath` to specify the specific file within the `ConfigMap` volume, we can selectively mount only the necessary configuration files or data that a container needs.

Apply the manifest to create the deployment:

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

Observe the `Mounts` field of the `nginx` container. You'll see 2 mounts from `my-volume`.


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

- The nginx server listens on port `8080`
- Nginx serves the supplied `index.html` page mounted at `/usr/share/nginx/html`

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

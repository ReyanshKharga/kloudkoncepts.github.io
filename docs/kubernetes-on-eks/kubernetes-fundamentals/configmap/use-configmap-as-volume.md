---
description: Unlock the potential of using ConfigMap as a volume in Kubernetes. Learn how to simplify configuration data handling by mounting it as a volume with our practical guide.
---

# Use ConfigMap as Volume

Let's see how we can use `ConfigMap` as a volume and mount it in a container.


## Step 1: Create a ConfigMap

=== ":octicons-file-code-16: `my-configmap.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-configmap
    data:
      # property-like keys; each key maps to a simple value
      foo1: bar1
      foo2: bar2

      # file-like keys; each key maps to a file content
      game.properties: |
        enemy.types=aliens,monsters
        player.maximum-lives=5    
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

Apply the manifest to create ConfigMap:

```
kubectl apply -f my-configmap.yml
```


## Step 2: Verify ConfigMap

```
# List configmaps
kubectl get cm

# Describe the configmap
kubectl describe cm my-configmap
```


## Step 3: Create Pods That Uses ConfigMap as Volume

Let's create pods that uses `ConfigMap` as volume and mounts it in a container. We'll use a deployment to create pods:


=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
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
              mountPath: "/config"
          volumes:
          - name: my-volume
            configMap:
              name: my-configmap
    ```

Observe the following:

- The pod uses the ConfigMap `my-configmap` as volume
- The volume is mounted at `/config` directory in the `nginx` container

Apply the manifest to create deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 4: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 5: Verify Volume Mount and Data

1. Open a shell session inside the `nginx` container:

    ```
    kubectl exec -it <pod-name> -- bash
    ```

2. View data:

    ```
    cd /config
    ls
    ```

Please note that when a `ConfigMap` is mounted as a volume in a container, each key in the `ConfigMap` is stored as a file in the container's file system. This means that the container can read the contents of each file as if they were regular files in the container's file system.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-configmap.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```
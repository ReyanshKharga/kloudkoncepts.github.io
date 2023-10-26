---
description: Efficiently create and manage ConfigMaps in Kubernetes with our step-by-step guide. Learn how to organize and utilize configuration data effectively for your applications. Start simplifying configuration management now!
---

# Create and Manage ConfigMaps

Let's see how we can create and manage `ConfigMaps`.


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


Observe the following:

- The `data` field is where we define key-value pairs
- There are two property-like keys `foo1`, and `foo2`
- There are two file-like keys `game.properties`, and `default.conf`


Apply the manifest to create the ConfigMap:

```
kubectl apply -f my-configmap.yml
```


## Step 2: List ConfigMaps

```
kubectl get configmaps
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get cm
kubectl get configmap
kubectl get configmaps
```

!!! note
    `configmap` is abbreviated as `cm`.


## Step 3: Describe a ConfigMap

```
# Command template
kubectl describe configmap <configmap-name>
{OR}
kubectl describe configmap/<configmap-name>

# Actual command
kubectl describe configmap my-configmap
{OR}
kubectl describe configmap/my-configmap
```

## Step 4: Delete a ConfigMap

```
# Command template
kubectl delete configmap <configmap-name>
{OR}
kubectl delete configmap/<configmap-name>

# Actual command
kubectl delete configmap my-configmap
{OR}
kubectl delete configmap/my-configmap
```

You can also use the manifest to delete the resource as follows:

```
kubectl delete -f my-configmap.yml
```
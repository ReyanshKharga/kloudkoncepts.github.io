---
description: Learn how to leverage Kubernetes Secrets as volumes for secure data storage in your applications. Discover the power of securely managing sensitive information with our step-by-step guide.
---

# Use Secret as Volume

Let's see how we can use `Secret` as a Volume and mount it in a container.


## Step 1: Create a Secret

Let's create a `Secret` that stores a certificate:

=== ":octicons-file-code-16: `my-secret.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Secret
    metadata:
      name: my-secret
    data:
      certificate.crt: |
        LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQW5LZ0F3SUJBZ0lKQUxkS2hN
        WGN3RlFZRFZRUURFeHpKYjNORWNtRnVhU0lzSW1saGRDSTZNVGMyT0RBeU1UQXcKT0RBME1E
        UXhNekl4TWpFd01USTVNRFl3T1RJeE1Gb1hEVEUzTVRjd09USTVNRFl3T1RJeE1Gb1hEVEUz
        TVRjd09USTVNRFl3TgpPVEk1TURZd09USTVNRFl3T1RJeE1Gb1hEVEUzTVRjd09USTVNRFl3
        T1RJeE1Gb1hEVEUzTVRjd09USTVNRFl3T1RJeE1Gb1hEVEUzTVRjd09USTVNRGIwCk1TMHdD
        d1lEVlFRREV4cEpiaTVqY21sbGR6RU5NQXNHQTFVRUF3d2FVSFZ5WTI5dE1CNFhEVEV4TUM0
        eEhUTXAKYldsdWJHbHVaR1Z5TG1OdmJRd3ZNQjRYRFRFNU1EQXdNRm9YRFRJNU1EWXhNVEF4
        TURBM01Gb1hEVEUzTVRjd09USTVNRFl3T1RJeE1GbwpPVEl3TVNJd0lBWURWUVFERXhwSmJp
        NWpjbmwwY21sdmJqMGlNUXd3RFFZSktvWklodmNOQVFFRkJRQURnZ0VOCkFEQ0NBUW9DZ2dF
        QkFIS2Q2NmxkMmM0THdEMjFrMzlDVk0wcGhXcXFLM3BmN2FzTVlVcEJyUGtqczczZUdIUGMK
        R3c4cHFuYWhhUDJibVRhZWR3SkhZcmM3dzNoMzRzRG5ER0tyUnp0ZlBkQmJUSURBUUFCbzJN
        QkVHQTFVZEV3RUIvdgpDQk1CZ0RBSUI2QXdnRUJBSlZYS2lHQ2RmN0dFQXR3RnBIZThFSlNa
        a2JNdmdrWnhWQlNUY0dIdW1YRUlBb0dBCkFBTU1rdDZJanNLQ0NnWUlLb1pJemowSUFRN0Jr
        QkRZ
    ```

Apply the manifest to create Secret:

```
kubectl apply -f my-secret.yml
```


## Step 2: Verify Secret

```
# List secrets
kubectl get secrets

# Describe the secret
kubectl describe secret my-secret
```


## Step 3: Create Pods That Uses Secret as Volume

Let's create pods that uses `Secret` as volume and mounts it in a container. We'll use a deployment to create pods:

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
            secret:
              secretName: my-secret
    ```

Observe the following:

- The pod uses the Secret `my-secret` as volume
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

1. Open a shell session inside the nginx container:

    ```
    kubectl exec -it <pod-name> -- bash
    ```

2. View data:

    ```
    cd /config
    ls
    cat certificate.crt
    ```

Please note that when a `Secret` is mounted as a volume in a container, each key in the `Secret` is stored as a file in the container's file system. This means that the container can read the contents of each file as if they were regular files in the container's file system.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-secret.yml
│   |-- my-deployment.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

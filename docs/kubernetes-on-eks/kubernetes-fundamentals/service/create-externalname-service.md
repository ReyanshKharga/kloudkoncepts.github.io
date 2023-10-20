---
description: Master the Art of Creating an ExternalName Service in Kubernetes. Explore the step-by-step process for setting up ExternalName services, providing seamless external DNS resolution for your applications within your Kubernetes cluster.
---

# Create ExternalName Service

An `ExternalName` service is a type of service in kubernetes that allows you to create a service that simply points to an external service by its DNS name, rather than routing traffic to a set of pods.

Let's see this in action!


## Step 1: Create ExternalName Service

Let's create an ExternalName service as follows:

=== ":octicons-file-code-16: `my-externalname-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-externalname-service
    spec:
      type: ExternalName
      externalName: google.com
    ```

Please note that we do not have `selectors` available for `ExternalName` services as they do not route traffic directly to pods but rather use an external DNS to resolve the service.

Let's apply the manifest to create the service:

```
kubectl apply -f my-externalname-service.yml
```


## Step 2: Verify the Service

```
# List services
kubectl get svc
```

Note the `EXTERNAL-IP` field in the output. You'll see the external DNS the service resolves to.


## Step 3: Access the Service

Let's see what happens when you access the service.

1. Create a simple pod that has ping Linux utility:

    ```
    kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
    ```

2. Start a shell session inside busybox container of the pod:

    ```
    kubectl exec -it busybox -- sh
    ```

3. Ping the service:

    ```
    ping my-externalname-service
    ```

You'll notice that it resolves to `google.com` which is the external DNS name configured for the service.


## Clean Up

```
# Delete service
kubectl delete -f my-externalname-service.yml

# Delete busybox pod
kubectl delete pod busybox
```

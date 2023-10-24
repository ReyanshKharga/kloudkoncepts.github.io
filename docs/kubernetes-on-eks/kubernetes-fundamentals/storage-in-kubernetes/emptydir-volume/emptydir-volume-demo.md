---
description: Explore emptyDir Volume in action with our hands-on demo. See how this Kubernetes storage feature works in real-life scenarios.
---

# emptyDir Volume Demo

Let's see how we can use `emptyDir` volume as temporary storage.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}


## Step 1: Create a Deployment With emptyDir Volume

Let's create a deployment as follows:

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
            image: reyanshkharga/nginx:v1
            command: ["/bin/sh"]
            args: ["-c", "while true; do echo $(date -u) >> /data/my-temp-data.txt; sleep 5; done"]
            volumeMounts:
            - name: my-volume
              mountPath: /data
          volumes:
          - name: my-volume
            emptyDir: {}
    ```

Observe the following:

1. We have added an `emptyDir` volume named `my-volume` in the pod template
2. The pod has only one container
3. The `emptyDir` volume is mounted on `/data` directory of the container
4. The container writes some data in the `my-temp-data.txt` file every 5 seconds

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 2: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: Verify Volume Mount and Data

Let's verify if the `emptyDir` volume was mounted in the container.

1. Start a shell session inside the container:

    ```
    kubectl exec -it <pod-name> -- bash
    ```

2. Verify if `/data` directory is present in the container:

    ```
    ls /data
    ```

3. View the content of `/data/my-temp-data.txt` file:

    ```
    tail -f /data/my-temp-data.txt
    ```

## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx
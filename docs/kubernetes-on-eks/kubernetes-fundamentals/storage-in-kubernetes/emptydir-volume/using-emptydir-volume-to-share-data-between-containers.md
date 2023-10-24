---
description: Unlock the power of emptyDir Volume to seamlessly share data between containers. Learn how to enhance container communication and efficiency with our step-by-step guide.
---

# Using emptyDir Volume to Share Data Between Containers

Let's see how we can use `emptyDir` volume to share data between containers.

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
          - name: writer
            image: reyanshkharga/nginx:v1
            command: ["/bin/sh"]
            args: ["-c", "while true; do echo $(date -u) >> /writer-data/my-data.txt; sleep 5; done"]
            volumeMounts:
            - name: my-volume
              mountPath: /writer-data
          - name: reader
            image: busybox
            command: ["/bin/sh"]
            args: ["-c", "while true; do tail -f /reader-data/my-data.txt; sleep 5; done"]
            volumeMounts:
            - name: my-volume
              mountPath: /reader-data
          volumes:
          - name: my-volume
            emptyDir: {}
    ```

Observe the following:

1. We have added an `emptyDir` volume in the pod template
2. The pod has two containers named writer and reader
3. The `emptyDir` volume is mounted on `/writer-data` directory of the writer container
4. The `emptyDir` volume is mounted on `/reader-data` directory of the reader container
5. The writer container writes some data in the `my-data.txt` file every 5 seconds
6. The reader container reads the same data from `my-data.txt` file every 5 seconds

You might wonder why the contents of the `/writer-data/my-data.txt` and `/reader-data/my-data.txt` files are the same.

The reason is that the `my-data.txt` file is stored in the shared `my-volume` volume, which is mounted to both containers on different directories. So any changes made to the file in one container are immediately visible in the other container, since they are both accessing the same file in the shared volume.

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 2: Verify Deployment and Pods

Let's verify if the `emptyDir` volume was mounted in both the containers.

1. Start a shell session inside the `writer` container:

    ```
    kubectl exec -it <pod-name> -c writer -- bash
    ```

2. Verify if `/writer-data` directory is present in the `writer` container:

    ```
    ls /writer-data
    ```

3. View the content of `/writer-data/my-data.txt` file:

    ```
    tail -f /writer-data/my-data.txt
    ```

4. Exit out of the `writer` container:

    ```
    exit
    ```

5. Start a shell session inside the `reader` container:

    ```
    kubectl exec -it <pod-name> -c reader -- sh
    ```

6. Verify if `/reader-data` directory is present in the `reader` container:

    ```
    ls /reader-data
    ```

7. View the content of `/reader-data/my-data.txt` file:

    ```
    tail -f /reader-data/my-data.txt
    ```

8. Exit out of the `reader` container:

    ```
    exit
    ```

9. View `reader` container logs:

    ```
    kubectl logs -f <pod-name> -c reader
    ```

Observations:

1. The content of `my-data.txt` is shared between `writer` and `reader` containers.
2. Any changes made to the `my-data.txt` file in the `writer` container are immediately visible in the `reader` container.
3. The `reader` container is able to read the shared data. (Verified by checking reader container logs)

!!! note
    The `busybox` image doesn't have `bash` so we have used `sh` instead to start a shell session inside `reader` container that uses the `busybox` image.

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
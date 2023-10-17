---
description: Discover the fundamentals of Busybox Pods in Kubernetes with our beginner's guide. Get started with understanding the purpose and functionality of Busybox Pods.
---


# Introduction to Busybox Pod

Let's learn about a pod called `busybox` that we will use a lot in this course for various purposes.

## Busybox Docker Image

- BusyBox image is a `Linux-based` image containing many handy utilities.
- The BusyBox container image is incredibly `lightweight`.
- The image is less than 3 MB in size.
- Busybox image doesn't have a `bash` shell. It has a shell at `/bin/sh`.


## Create Busybox Pod

=== ":octicons-file-code-16: `busybox.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        args:
          - /bin/sh
          - -c
          - sleep 3600
      restartPolicy: Never
    ```

```
# Create busybox pod
kubectl apply -f busybox.yml
```

- The busybox container has an entrypoint `/bin/sh` and it has no running tasks. The process comes to an end.
- In order to keep the container alive we are making it sleep for 3600 seconds.

You can also use the following imperative command to create a busybox pod:

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
```

## Clean Up

Let's delete the busybox Pod.

```
kubectl delete -f busybox.yml
```
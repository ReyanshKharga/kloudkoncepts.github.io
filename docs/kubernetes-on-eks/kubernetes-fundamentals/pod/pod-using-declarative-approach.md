---
description: Learn how to create and manage kubernetes pods using a declarative approach. Discover the power of declarative configurations for efficient management of your containerized applications.
---

# Create and Manage Kubernetes Pods Using Declarative Approach

Let's see how you can create and manage kubernetes Pods declaratively.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}


## Step 1: Create Pod Manifest

First, we need to write the pod manifest as follows:

=== ":octicons-file-code-16: `my-pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
    name: my-pod
    spec:
    containers:
    - name: my-container
      image: reyanshkharga/nginx:v1
      imagePullPolicy: Always
      ports:
      - containerPort: 80
    ```

Required fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a name string, UID , and - optional namespace.
- `spec` - What state you desire for the object.


## Step 2: Create Pod

Let's use `kubectl apply` to apply the manifest and create the pod:

```
# Create pod
kubectl apply -f my-pod.yml
```

## Step 3: List Pods

List pods and verify that the pod we created is up and running:

```
kubectl get pods
```

## Step 4: Verify Application

Verify that the application in the container inside the pod is running properly:

```
# Start shell session inside the container
kubectl exec -it my-pod -- curl localhost

# Access application
curl localhost
```

Also, let's use `kubectl port-forward` to access the application locally:

```
kubectl port-forward my-pod 5000:80
```

Open any browser and visit `localhost:5000`.


## Step 5: Delete Pod

```
# Delete the pod
kubectl delete -f my-pod.yml
```

!!! quote "References:"
    !!! quote ""
        * [Pods Concept]{:target="_blank"}
        * [Pod v1 core]{:target="_blank"}
        * [Workload Resources - Pod]{:target="_blank"}
        * [ObjectMeta]{:target="_blank"}
        * [PodSpec]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx
[Pods Concept]: https://kubernetes.io/docs/concepts/workloads/pods/
[Pod v1 core]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#pod-v1-core
[Workload Resources - Pod]: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/
[ObjectMeta]: https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/object-meta/#ObjectMeta
[PodSpec]: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec
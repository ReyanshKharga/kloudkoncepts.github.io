---
description: Discover how to create and manage Kubernetes namespaces using a declarative approach. Streamline your container organization with best practices.
---


# Create and Manage Namespaces Using Declarative Approach

Let's see how you can create and manage kubernetes namespaces declaratively.


## Step 1: Create a Namespace

Let's create two namespaces `dev` and `prod` as follows:

=== ":octicons-file-code-16: `dev-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ```

=== ":octicons-file-code-16: `prod-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: prod
    ```

Apply the manifest files to create namespaces:

```
# Create dev namespace
kubectl apply -f dev-namespace.yml

# Create prod namespace
kubectl apply -f prod-namespace.yml
```


## Step 2: List Namespaces

List all the namespaces:

```
kubectl get namespaces
```


## Step 3: Describe a Namespace

Describe `dev` namespace:

```
kubectl describe namespace dev
```

Describe `prod` namespace:

```
kubectl describe namespace prod
```


## Step 4: Create Kubernetes Resources in a Namespace

Let's create two pods. One in `dev` namespace and another in `prod` namespace.

=== ":octicons-file-code-16: `dev-pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: dev-pod
      namespace: dev
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: Always
        ports:
        - containerPort: 80
    ```

=== ":octicons-file-code-16: `prod-pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: prod-pod
      namespace: prod
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: Always
        ports:
        - containerPort: 80
    ```

Apply the manifest files to create pods:

```
# Create dev-pod
kubectl apply -f dev-pod.yml

# Create prod-pod
kubectl apply -f prod-pod.yml
```

Similarly you can create `Deployment`, `ReplicaSet`, `Service` etc. in a namespace. You just have to specify namespace in the metadata section of the resource definition.

Resources will be configured in the `default` namespace if the `namespace` field is not set in the `metadata` section of the resource definition.


## Step 5: List Resources in a Namespace

List all the pods in the `dev` namespace:

```
kubectl get pods -n dev
```

List all the pods in the `prod` namespace:

```
kubectl get pods -n prod
```

Also, let's start a shell session inside the `nginx` container of both the pods to verify if the application is running as expected.

1. Verify nginx container of `dev-pod` in the `dev` namespace:

    ```
    # Start a shell session
    kubectl exec -it dev-pod -n dev -- bash

    # Get the default nginx page
    curl localhost
    ```

2. Verify nginx container of `prod-pod` in the `prod` namespace:

    ```
    # Start a shell session
    kubectl exec -it prod-pod -n prod -- bash

    # Get the default nginx page
    curl localhost
    ```


## Step 6: Delete a Resource in a Namespace

Delete `dev-pod` from `dev` namespace:

```
kubectl delete pod dev-pod -n dev
{OR}
kubectl delete -f dev-pod.yml
```

Delete `prod-pod` from `prod` namespace:

```
kubectl delete pod prod-pod -n prod
{OR}
kubectl delete -f prod-pod.yml
```


## Step 7: Delete a Namespace

Delete `dev` namespace:

```
kubectl delete ns dev
{OR}
kubectl delete -f dev-namespace.yml
```

Delete `prod` namespace:

```
kubectl delete ns prod
{OR}
kubectl delete -f prod-namespace.yml
```
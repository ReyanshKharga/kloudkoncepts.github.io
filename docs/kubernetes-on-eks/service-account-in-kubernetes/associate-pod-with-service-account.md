---
description: Learn how to associate pods with Service Accounts in Kubernetes. Discover how this connection enhances security and identity management for your workloads. Dive into Pod with Service Account details today!
---

# Associate Pod With Service Account

Let's see how `service account` is used to grant pods the permissions to call kubernetes API and manage resources.


## Step 1: Create a Pod

Let's create pods. We'll use deployment to create pods:

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
    ```


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


## Step 3: Describe Pod

```
kubectl describe pod <pod-name>
```

You'll notice the pod has the `default` service account assigned to it even though we didn't specify it explicitly.

Now, let's take a look at the volume mounts.

```
kubectl get pod <pod-name> -o yaml
```

You'll observe that `token`, `ca.crt`, and `namespace` files are mounted in the pod at `/var/run/secrets/kubernetes.io/serviceaccount`.

The tokens are obtained directly through the `TokenRequest` API, and are mounted at the path `/var/run/secrets/kubernetes.io/serviceaccount` into pods using a projected volume. Also, these tokens will be automatically invalidated when their associated pod is deleted.

!!! note
    A `projected` volume is a volume that projects several existing volume sources into the same directory.

By having the service account and token available within the pod, you can utilize them to securely communicate with the kubernetes API or other services that require authentication within the cluster.


## Step 4: Verify Volume Mounts

```
# Start a shell session inside the pod
kubectl exec -it <pod-name> -- bash

# List mounted files
ls /var/run/secrets/kubernetes.io/serviceaccount
```

You'll see the `ca.crt`, `namespace`, and `token` files as discussed in the previous step.

- `ca.cert`: The trust anchor needed to validate the certificate presented by the kubernetes API Server in this cluster.
- `namespace`: Namespace the secret is in.
- `token`: A bearer token that you can attach to API requests. This has a structure of a JSON Web Token (JWT).


## Step 5: Decode Certificate File

Let's decode the `ca.crt` file:

```
# Get the content of ca.crt
cat ca.crt
```

Copy the content of the `ca.crt`.

Visit [SSL Certificate Decoder]{:target="_blank"} and paste the content of certificate. Next, click on decode.

You'll see the detailed information about the certificate.


## Step 6: Decode Token File

Let's decode the `token` file:

```
# Get the content of token
cat token
```

Copy the content of the `token`.

Visit [jwt.io]{:target="_blank"} and paste the content of `token`.

You'll see the detailed information about the `token` such as expiry (exp), issued at time (iat), issuer (iss), subject (sub) and other details.


## Step 7: Access the Kubernetes API

List kubernetes services:

```
kubectl get svc
```

You'll see a service named `kubernetes`.

The `kubernetes` service in the `default` namespace is a service which forwards requests to the kubernetes master ( Typically kubernetes API server).

Now, let's start a shell session inside the container and try to access kubernetes API:

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# Change working directory
cd /var/run/secrets/kubernetes.io/serviceaccount

# Store the token in a variable
TOKEN=$(cat token)

# Verify the content of TOKEN variable
echo $TOKEN
```

Call the Kubernetes API:

```
curl https://kubernetes/api --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

A response from the kubernetes API server is returned with status code `200`.

Now, call the `v1` API:

```
curl https://kubernetes/api/v1 --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

Again a response from the Kubernetes API server is returned with status code `200`.


Call the `pods` API:

```
curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

A response from the kubernetes API server is returned with status code `403 (Forbidden)` and an error message that says:

```
"pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\""
```

This is because the service account (the identity) the pod uses doesn't have permission to call the pods api.

Here are the steps you can follow to assign permissions to a service account:

- Create a role using kubernetes `Role` object
- Bind the service account to the `Role` using kubernetes `RoleBinding` object

Since `default` service account is assigned to each pod that is created when you don't provide the service account name, modifying the permission of this service account is not recommended.

We'll create a new service account and then modify permission of that service account.


Let's clean up first:

```
kubectl delete -f my-deployment.yml
```


## Step 8: Create Service Account with Permission to Call Pods API

Let's create a `service account` that has permission to call the kubernetes pods API and then assign this service account to a pod.

### 1. Create a Service Account

=== ":octicons-file-code-16: `my-service-account.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: my-service-account
    ```

Apply the manifest to create the service account:

```
kubectl apply -f service-account.yml
```

Verify the service account:

```
kubectl get sa
```


### 2. Create Pods

Let's create pods that use the service account we created in the previous step. We'll use deployment to create pods:

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
          serviceAccountName: my-service-account
          containers:
          - name: nginx
            image: nginx
    ```

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```

Verify the deployment and pods:

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods

# Describe pod
kubectl describe pod <pod-name>
```

You'll notice that `my-service-account` is assigned to the pods.


### 3. Try to Access the Kubernetes API

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# Change working directory
cd /var/run/secrets/kubernetes.io/serviceaccount

# Store the token in a variable
TOKEN=$(cat token)

# Verify the content of TOKEN variable
echo $TOKEN
```


Call the Kubernetes API:

```
curl https://kubernetes/api --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

A response from the kubernetes API server is returned with status code `200`.

Call the `v1` API:
```
curl https://kubernetes/api/v1 --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

Again a response from the Kubernetes API server is returned with status code `200`.

Call the `pods` API:

```
curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

A response from the kubernetes `API server` is returned with status code `403 (Forbidden)` and an error message that says:

```
"pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\""
```

This is because the service account still doesn't have the permission to call the `pods` API.


### 4. Create a Role

Let's create a role that has permissions to call the pods API. For this, we need to create a `Role` kubernetes object as follows:

=== ":octicons-file-code-16: `my-role.yml`"

    ```yaml linenums="1"
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: my-role
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    ```

Apply the manifest to create the role:

```
kubectl apply -f my-role.yml
```

Verify the role:

```
# List roles
kubectl get roles

# Describe the role
kubectl describe role my-role
```

### 5. Bind the Role to the Service Account

Now, let's bind the role to the service account we created. For this, we need to create a `RoleBinding` kubernetes object as follows:

=== ":octicons-file-code-16: `my-rolebinding.yml`"

    ```yaml linenums="1"
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: my-rolebinding
    subjects:
      - kind: ServiceAccount
        name: my-service-account
    roleRef:
      kind: Role
      name: my-role
      apiGroup: rbac.authorization.k8s.io
    ```

Apply the manifest to create the role binding:

```
kubectl apply -f my-rolebinding.yml
```

Verify the role bindings:

```
# List rolebindings
kubectl get rolebindings

# Describe the rolebinding
kubectl describe rolebinding my-rolebinding
```

### 6. Access the Kubernetes API Again

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# Change working directory
cd /var/run/secrets/kubernetes.io/serviceaccount

# Store the token in a variable
TOKEN=$(cat token)

# Verify the content of TOKEN variable
echo $TOKEN
```

Call the Kubernetes API:
```
curl https://kubernetes/api --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

A response from the kubernetes API server is returned with status code `200`.


Call the v1 API:
```
curl https://kubernetes/api/v1 --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

Again a response from the Kubernetes API server is returned with status code `200`.

Call the pods API:

```
curl https://kubernetes/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --cacert ./ca.crt
```

This time you will get a response with status code `200`. This is because the `service account` the pods use has access to the pods API.

Here's a pictorial representation of how pods use a service account to access kubernetes resources:

<p align="center">
    <img src="../../../../assets/eks-course-images/service-account/pod-with-service-account.png" alt="Liveness Probe" loading="lazy" width="550" />
</p>


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service-account.yml
│   |-- my-role.yml
│   |-- my-rolebinding.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

!!! quote "References:"
    !!! quote ""
        * [SSL Certificate Decoder]{:target="_blank"}
        * [jwt.io]{:target="_blank"}


<!-- Hyperlinks -->
[SSL Certificate Decoder]: https://www.sslchecker.com/certdecoder
[jwt.io]: https://jwt.io/
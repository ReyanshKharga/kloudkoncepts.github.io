---
description: Learn how to Create and Manage Kubernetes Services effortlessly. Get hands-on with step-by-step guides and best practices for creating, configuring, and maintaining services within your Kubernetes environment. Master the art of service management in Kubernetes with our comprehensive resource.
---

# Create and Manage Kubernetes Service

From this topic onwards we won't be using imperative commands to create Kubernetes resources unless required.

Kubernetes imperative commands are useful for quickly creating and managing simple Kubernetes resources, but they can become less efficient and more error-prone for complex objects like services.

## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/nginx:v1]{:target="_blank"}
- [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1] runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Step 1. Create a Deployment

First, we need a set of pods that we want to expose using a service.

So, let's create a deployment as follows:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nodeapp
      template:
        metadata:
          labels:
            app: nodeapp
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 5000
    ```

```
# Create deployment
kubectl apply -f my-deployment.yml
```


## Step 2: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: Create Service Manifest

First, we need to write the service manifest as follows:

=== ":octicons-file-code-16: `my-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      type: ClusterIP
      selector:
        app: nodeapp
      ports:
        - port: 80
          targetPort: 5000
    ```

The `port` field defines the port number on which the service will listen for incoming traffic.

The `targetPort` field defines the port number on which the pods associated with the service are listening for incoming traffic.

Required fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a name string, UID , and optional namespace.
- `spec` - What state you desire for the object.


## Step 4: Create Service

Let's use `kubectl apply` to apply the manifest and create the service:

```
# Create service
kubectl apply -f my-service.yml
```


## Step 5: List Services

```
# List all services
kubectl get services

# List all services with expanded (aka "wide") output
kubectl get services -o wide
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get svc 
kubectl get service
kubectl get services
```

!!! note
    `service` is abbreviated as `svc`.


You'll notice the following fields:

- `TYPE`: Service type
- `CLUSTER-IP`: Virtual IP address assigned to a Service in Kubernetes, which is used for communication within the cluster.
- `EXTERNAL-IP`: The IP address or Load Balancer DNS name that external clients can use to access the Service from outside of the cluster. (Applicable only for `LoadBalancer` Service)
- `PORT(S)`: The ports that the service is listening on

You'll also notice a service with name kubernetes which is a built-in service in kubernetes that is automatically created when you deploy a kubernetes cluster. It is a special service that provides a way for the various components of the kubernetes control plane to communicate with each other.

In general, you don't need to interact with the kubernetes service directly, as it is managed by the kubernetes control plane. However, it is an important component of a kubernetes cluster, as it provides a way for the various components of the control plane to work together and manage the cluster.


## Step 6: Access Service Locally

As the service is configured as `ClusterIp`, it cannot be accessed externally. Hence, we require a proxy to access it.

Let's use `kubectl port-forward` command to access the service.

The `kubectl port-forward` command creates a proxy that allows you to access a resource in a kubernetes cluster from your local machine.

The following command forwards traffic from the local port `8080` to port `80` on the service. You can replace the local port number with any available port on your local machine.

```
kubectl port-forward svc/my-service 8080:80
```

Open any browser on your local machine and access `localhost:8080`.

!!! note "Important Note"
    When you use the `kubectl port-forward` command to access a kubernetes service, by default it will forward traffic to a single pod selected by the kubernetes service's load-balancing algorithm. This means that if the service has multiple pods backing it, the `kubectl port-forward` command will only forward traffic to one of the pods.

    This behavior is by design to prevent multiple users from interfering with each other's port forwarding sessions.


## Step 7: Access Service From a Pod in the Cluster

Let's see how to access the service from another pod in the cluster.

1. Create a simple pod:

    ```
    kubectl run nginx --image=reyanshkharga/nginx:v1
    ```

2. Start a shell session inside nginx container of the pod:

    ```
    kubectl exec -it nginx -- bash
    ```

Now, there are two ways to access a kubernetes service:


1. Using the Service's Cluster IP:

    This method allows other resources within the kubernetes cluster to access the service using its Cluster IP address.

    ```
    # Access the service using cluster IP
    curl <cluster-ip-of-service>:<service-port>
    ```

2. Using the Service's Fully Qualified Domain Name (FQDN):

    This method allows resources outside the kubernetes cluster to access the service using its FQDN, which is typically a combination of the service name and the cluster's domain name.

    ```
    # Know the cluster domain
    kubectl get cm coredns -n kube-system -o jsonpath='{.data.Corefile}' | grep "kubernetes" | awk '{print $2}'
    ```

    The domain name `cluster.local` is commonly used as the default cluster domain name in kubernetes installations.

    ```
    # Command template to access the service using FQDN
    curl <service-name>.<namespace>.svc.<cluster-domain>:<service-port>

    # Actual command
    curl my-service.default.svc.cluster.local:80
    ```

Hit the service a few times and you'll see the traffic being served from the pods the service manages. You'll also notice that the service performs load balancing for the traffic.


## Step 8: Delete the Service

Let's delete the service:

```
kubectl delete -f manifests/my-service.yml
{OR}
kubectl delete svc/my-service
{OR}
kubectl delete svc my-service
```


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
```

You can delete all the kubernetes objects we created in one go:

```
kubectl delete -f manifests/
```




!!! quote "References:"
    !!! quote ""
        * [Service v1 core]{:target="_blank"}
        * [Service Resources - Service]{:target="_blank"}
        * [ObjectMeta]{:target="_blank"}
        * [ServiceSpec]{:target="_blank"}
        * [Service Concept]{:target="_blank"}




<!-- Hyperlinks -->
[reyanshkharga/nginx:v1]: https://hub.docker.com/r/reyanshkharga/nginx
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[Service v1 core]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#service-v1-core
[Service Resources - Service]: https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/
[ObjectMeta]: https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/object-meta/#ObjectMeta
[ServiceSpec]: https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/#ServiceSpec
[Service Concept]: https://kubernetes.io/docs/concepts/services-networking/service/
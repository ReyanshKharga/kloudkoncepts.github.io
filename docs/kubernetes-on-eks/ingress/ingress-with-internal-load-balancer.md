---
description: Manage Kubernetes Ingress easily. Explore our guide on creating Ingress with an Internal Load Balancer. Learn how to set up internal load balancing for your applications in a straightforward way.
---

# Create Ingress With Internal Load Balancer

You can create an internal load balancer to distribute traffic to your EC2 instances from clients with access to the VPC for the load balancer.

An internal load balancer routes requests to targets using private IP addresses.

You can set `alb.ingress.kubernetes.io/scheme` to `internal` to instruct AWS Load Balancer Controller to create an internal application load balancer.

Let's see this in action!


## Docker Images

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1]{:target="_blank"} runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Step 1: Create a Deployment

First, let's create a deployment as follows:

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
          app: demo
      template:
        metadata:
          labels:
            app: demo
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 5000
    ```

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```

Verify deployment and pods:

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 2: Create a Service

Next, let's create a service as follows:

=== ":octicons-file-code-16: `my-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeport-service
    spec:
      type: NodePort
      selector:
        app: demo
      ports:
        - port: 5000
          targetPort: 5000
    ```

Apply the manifest to create the service:

```
kubectl apply -f my-service.yml
```

Verify service:

```
kubectl get svc
```


## Step 3: Create Ingress

Now that we have the service ready, let's create an Ingress object that creates an internal load balancer:

=== ":octicons-file-code-16: `my-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internal # Default value is internal
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
        alb.ingress.kubernetes.io/target-type: instance # Default is instance
    spec:
      ingressClassName: alb
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-nodeport-service
                port:
                  number: 5000
    ```

Note that we have set the value of `alb.ingress.kubernetes.io/scheme` to `internal` so that the Load Balancer Controller creates an internal load balancer.

Apply the manifest to create ingress:

```
kubectl apply -f my-ingress.yml
```

Verify ingress:

```
kubectl get ingress
{OR}
kubectl get ing
```


## Step 4: Verify AWS Resources in AWS Console

Visit the AWS console and verify the resources created by AWS Load Balancer Controller.

Pay close attention to the type of load balancer. It should be `internal`.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 5: Access App Using Internal Load Balancer DNS

Because the load balancer is internal, access to our app from outside the VPC is restricted. To overcome this, let's create a pod that we can use to access the load balancer and, in turn, our app. Since the pod will reside within the same VPC, we will be able to access our app.

First, let's create a pod as follows:

=== ":octicons-file-code-16: `nginx-pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
    ```

Apply the manifest to create the pod:

```
kubectl apply -f nginx-pod.yml
```

Now, let's start a shell session inside the nginx container and hit the internal load balancer url:

```
# Start a shell session inside the nginx container
kubectl exec -it nginx -- bash

# Hit the load balancer url using CURL
curl <internal-alb-dns>
```

You'll see the response from the app.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
│   |-- my-ingress.yml
│   |-- nginx-pod.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
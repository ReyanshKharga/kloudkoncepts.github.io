---
description: Efficiently route traffic to your Kubernetes services with our guide on creating Ingress with Path-Based Routing. Learn how to tailor your application's traffic paths and improve user experience. Get started with Path-Based Routing and optimize your Kubernetes deployment.
---

# Create Ingress With Path Based Routing

Path-based routing is one of the features offered by Application Load Balancer.

The Application Load Balancer forwards the requests to the specific targets based on the rules configured in the load balancer.


## Overriding Health Check Configuration

Since each target in this case might have different health check requirements, you can override the ingress health check configurations by adding the same health check annotations in kubernetes service definition.


## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/reactapp:profile]{:target="_blank"}
- [reyanshkharga/reactapp:notifications]{:target="_blank"}
- [reyanshkharga/reactapp:feed]{:target="_blank"}

!!! note
    All microservices (`profile`, `notifications`, and `feed`) run on port `3000`.

    - [reyanshkharga/reactapp:profile]{:target="_blank"} is a react app that serves request on `/profile` path.
    - [reyanshkharga/reactapp:notifications]{:target="_blank"} is a react app that serves request on `/notifications` path.
    - [reyanshkharga/reactapp:feed]{:target="_blank"} is a react app that serves request on `/feed` path.


## Objective

In this example we will have 3 microservices:

1. `profile`: uses docker image `reyanshkharga/reactapp:profile`
2. `notifications`: uses docker image `reyanshkharga/reactapp:notifications`
3. `feed`: uses docker image `reyanshkharga/reactapp:feed`

We'll do the following:

1. Create a deployment and service for `profile` microservice.
2. Create a deployment and service for `notifications` microservice.
3. Create a deployment and service for `feed` microservice.
4. Create a ingress that sends traffic to one of the microservices based on the path.
5. We'll also provide separate health check path for each microservice using `alb.ingress.kubernetes.io/healthcheck-path` annotation in the service definition of each microservice.



## Step 1: Create Kubernetes Objects

Let's create the kubernetes objects as discussed above:

=== ":octicons-file-code-16: `profile.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: profile-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: profile
      template:
        metadata:
          labels:
            app: profile
        spec:
          containers:
          - name: profile-container
            image: reyanshkharga/reactapp:profile
            imagePullPolicy: Always
            ports:
              - containerPort: 3000
    # Service
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: profile-nodeport-service
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /profile
    spec:
      type: NodePort
      selector:
        app: profile
      ports:
        - port: 3000
          targetPort: 3000
    ```

=== ":octicons-file-code-16: `notifications.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: notifications-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: notifications
      template:
        metadata:
          labels:
            app: notifications
        spec:
          containers:
          - name: notifications-container
            image: reyanshkharga/reactapp:notifications
            imagePullPolicy: Always
            ports:
              - containerPort: 3000
    # Service
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: notifications-nodeport-service
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /notifications
    spec:
      type: NodePort
      selector:
        app: notifications
      ports:
        - port: 3000
          targetPort: 3000
    ```

=== ":octicons-file-code-16: `feed.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: feed-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: feed
      template:
        metadata:
          labels:
            app: feed
        spec:
          containers:
          - name: feed-container
            image: reyanshkharga/reactapp:feed
            imagePullPolicy: Always
            ports:
              - containerPort: 3000
    # Service
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: feed-nodeport-service
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /feed
    spec:
      type: NodePort
      selector:
        app: feed
      ports:
        - port: 3000
          targetPort: 3000
    ```

=== ":octicons-file-code-16: `ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        # Load Balancer Annotations
        alb.ingress.kubernetes.io/scheme: internet-facing # Default value is internal
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
        # Health Check Annotations
        alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
        alb.ingress.kubernetes.io/healthcheck-port: traffic-port
        alb.ingress.kubernetes.io/healthcheck-path: /health
        alb.ingress.kubernetes.io/healthcheck-interval-seconds: '5'
        alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '2'
        alb.ingress.kubernetes.io/success-codes: '200'
        alb.ingress.kubernetes.io/healthy-threshold-count: '2'
        alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'
    spec:
      ingressClassName: alb
      rules:
      - http:
          paths:
          - path: /profile
            pathType: Prefix
            backend:
              service:
                name: profile-nodeport-service
                port:
                  number: 3000
      - http:
          paths:
          - path: /notifications
            pathType: Prefix
            backend:
              service:
                name: notifications-nodeport-service
                port:
                  number: 3000
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: feed-nodeport-service
                port:
                  number: 3000
    ```

Observe the following:

1. We've provided the health check paths for each of the microservices using the `alb.ingress.kubernetes.io/healthcheck-path` annotation.
2. In the ingress, we have used path-based rules to route traffic to a particular microservice based on matching path.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- profile.yml
│   |-- notifications.yml
│   |-- feed.yml
│   |-- ingress.yml
```

Let's apply the manifests to create the kubernetes objects:

```
kubectl apply -f manifests/
```

This will create the following resources:

- Deployment and service for `profile` microservice.
- Deployment and service for `notifications` microservice.
- Deployment and service for `feed` microservice.
- Ingress with three rules.


## Step 2: Verify Kubernetes Objects

```
# List pods
kubectl get pods

# List deployments
kubectl get deployments

# List services
kubectl get svc

# List ingress
kubectl get ingress
```

Also, go to the AWS Console and verify the resources created by the AWS Load Balancer Controller, including the load balancer, target groups, listener rules, etc.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 3: Access App Via Load Balancer DNS

Copy the load balancer DNS from the ingress or AWS console and verify the following paths:

```
<alb-dns>/profile
<alb-dns>/notifications
<alb-dns>/feed
```

If everything is fine, the `<alb-dns>/profile`, `<alb-dns>/notifications`, and `<alb-dns>/feed` paths should serve the `profile`, `notifications`, and `feed` microservices, respectively.


## Ordering of Load Balancer Listener Rules

Rules are evaluated in priority order, from the lowest value to the highest value. The default rule is evaluated last. You can change the priority of a non-default rule at any time but you cannot change the priority of the default rule.

In later sections, we'll explore how to use the `IngressGroup` feature of the AWS Load Balancer Controller to specify the desired ordering of listener rules.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- profile.yml
│   |-- notifications.yml
│   |-- feed.yml
│   |-- ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


!!! quote "References:"
    !!! quote ""
        * [Path-Based Routing on an Application Load Balancer]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/reactapp:profile]: https://hub.docker.com/r/reyanshkharga/reactapp
[reyanshkharga/reactapp:notifications]: https://hub.docker.com/r/reyanshkharga/reactapp
[reyanshkharga/reactapp:feed]: https://hub.docker.com/r/reyanshkharga/reactapp
[Path-Based Routing on an Application Load Balancer]: https://aws.amazon.com/premiumsupport/knowledge-center/elb-achieve-path-based-routing-alb/
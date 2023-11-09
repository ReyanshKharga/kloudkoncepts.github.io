---
description: Simplify Kubernetes Ingress. Learn how to create Ingress with IngressGroup to better manage your applications. Organize and streamline your Ingress resources easily. Improve your application's scalability with IngressGroup.
---

# Create Ingress With IngressGroup

`IngressGroup` feature enables you to group multiple Ingress resources together. The controller will automatically merge Ingress rules for all Ingresses within `IngressGroup` and support them with a single ALB.

The <a>`alb.ingress.kubernetes.io/group.name`</a> annotation specifies the group name that this Ingress belongs to.

The <a>`alb.ingress.kubernetes.io/group.order`</a> annotation specifies the order across all Ingresses within `IngressGroup`.

By default, Ingresses don't belong to any `IngressGroup`, and we treat it as a "implicit IngressGroup" consisting of the Ingress itself.

Ingresses with same `group.name` annotation will form an "explicit IngressGroup".

Rules with the same order are sorted lexicographically by the Ingress’s namespace/name.

!!! warning
    If you turn your Ingress to belong a "explicit IngressGroup" by adding `group.name` annotation, other kubernetes users may create/modify their Ingresses to belong to the same `IngressGroup`, and can thus add more rules or overwrite existing rules with higher priority to the ALB for your Ingress.


## Prerequisite

To follow this tutorial, you'll require a domain and, additionally, an SSL certificate for the domain and its subdomains.

1. Register a Route 53 Domain

    Go to AWS Console and register a Route 53 domain. You can opt for a cheaper TLD (top level domain) such as `.link`

    !!! note
        It usually takes about 10 minutes but it might take about an hour for the registered domain to become available.

2. Request a Public Certificate

    Visit AWS Certificate Manager in AWS Console and request a public certificate for your domain and all the subdomains. For example, if you registered for a domain `example.com` then request certificate for `example.com` and `*.example.com`

    !!! note
        Make sure you request the certificate in the region where your EKS cluster is in.

3. Validate the Certificate

    Validate the requested certificate by adding `CNAME` records in Route 53. It is a very simple process. Go to the certificate you created and click on `Create records in Route 53`. The `CNAMEs` will be automatically added to Route 53.

    !!! note
        It usually takes about 5 minutes but it might take about an hour for the certificate to be ready for use.


Now that you have everything you need, let's move on to the demonstration.


## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/reactapp:v1]{:target="_blank"}
- [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1]{:target="_blank"} runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10

    [reyanshkharga/reactapp:v1]{:target="_blank"} is a frontend app that runs on port `3000`.


## Objective

In this example we will have 2 microservices:

1. `backend`: uses docker image `reyanshkharga/nodeapp:v1`
2. `frontend`: uses docker image `reyanshkharga/reactapp:v1`

We'll do the following:

1. Create a deployment and service for `backend` microservice.
2. Create a deployment and service for `frontend` microservice.
3. Create a ingress with ingress group and order for `backend` microservice.
4. Create a ingress with ingress group and order for `frontend` microservice.


## Step 1: Create Kubernetes Objects

Let's create the kubernetes objects as discussed above:

=== ":octicons-file-code-16: `backend.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: backend-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: backend
      template:
        metadata:
          labels:
            app: backend
        spec:
          containers:
          - name: backend-container
            image: reyanshkharga/nodeapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 5000
    # Service
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: backend-nodeport-service
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /health
    spec:
      type: NodePort
      selector:
        app: backend
      ports:
        - port: 5000
          targetPort: 5000
    ```

=== ":octicons-file-code-16: `frontend.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: frontend
      template:
        metadata:
          labels:
            app: frontend
        spec:
          containers:
          - name: frontend-container
            image: reyanshkharga/reactapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 3000
    # Service
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: frontend-nodeport-service
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /
    spec:
      type: NodePort
      selector:
        app: frontend
      ports:
        - port: 3000
          targetPort: 3000
    ```

=== ":octicons-file-code-16: `backend-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: backend-ingress
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
        # SSL Annotations
        alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-2016-08 # Optional
        # Listerner Ports Annotation
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
        # SSL Redicrect Annotation
        alb.ingress.kubernetes.io/ssl-redirect: '443'
        # IngressGroup
        alb.ingress.kubernetes.io/group.name: my-group
        alb.ingress.kubernetes.io/group.order: '1'
    spec:
      ingressClassName: alb
      rules:
      - host: api.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-nodeport-service
                port:
                  number: 5000
    ```


=== ":octicons-file-code-16: `frontend-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: frontend-ingress
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
        # SSL Annotations
        alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-2016-08 # Optional
        # Listerner Ports Annotation
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
        # SSL Redicrect Annotation
        alb.ingress.kubernetes.io/ssl-redirect: '443'
        # IngressGroup
        alb.ingress.kubernetes.io/group.name: my-group
        alb.ingress.kubernetes.io/group.order: '0'
    spec:
      ingressClassName: alb
      rules:
      - host: app.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-nodeport-service
                port:
                  number: 3000
    ```

Notice that we have defined the `group.name` and `group.order` annotations for both the ingress. Having same `group.name` will ensure that a single load balancer is created and shared by both the ingress.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- backend.yml
│   |-- frontend.yml
│   |-- backend-ingress.yml
│   |-- frontend-ingress.yml
```

Let's apply the manifests to create the kubernetes objects:

```
kubectl apply -f manifests/
```

This will create the following resources:

- Deployment and service for `backend` microservice.
- Deployment and service for `frontend` microservice.
- Ingress for `backend` and `frontend` microservices.


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

Go to the AWS Console and verify the resources created by the AWS Load Balancer Controller, including the load balancer, target groups, listener rules, etc.

You will observe that only one load balancer was created with two rules, following the ordering defined by the `group.order` annotation in the ingress.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 3: Add Records in Route 53

Go to AWS Route 53 and add two `A` records (`api.example.com` and `app.example.com`) that points to the load balancer that was created. You can use alias to point the subdomain to the load balancer.


## Step 4: Access App Using Route 53 DNS

Once the load balancer is in `Active` state, you can hit the subdomains you created in Route 53 and verify if everything is working properly.

Try accessing the following hosts:

```
# Backend
https://api.example.com

# Frontend
https://app.example.com
```

Also, verify that `HTTP` is redirected to `HTTPS`.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- backend.yml
│   |-- frontend.yml
│   |-- backend-ingress.yml
│   |-- frontend-ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

Also, go to Route 53 and delete the `A` records that you created.


!!! quote "References:"
    !!! quote ""
        * [IngressGroup]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[reyanshkharga/reactapp:v1]: https://hub.docker.com/r/reyanshkharga/reactapp
[IngressGroup]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/#ingressgroup
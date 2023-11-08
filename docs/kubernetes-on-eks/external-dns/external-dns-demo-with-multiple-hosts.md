---
description: See ExternalDNS in action. Watch our demo of ExternalDNS with multiple hosts. Learn how this tool helps you make Kubernetes resources discoverable with public DNS servers.
---

# ExternalDNS Demo With Multiple Hosts

Simply add the `external-dns.alpha.kubernetes.io/hostname` annotation to either the kubernetes Ingress or Service, and ExternalDNS will use this information to create corresponding Route 53 records.

Let's see this in action!


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
3. Create a ingress that sends traffic to one of the microservices based on the host.
4. We'll also provide separate ExternalDNS configuration for each microservice using `external-dns.alpha.kubernetes.io/hostname` annotation in the service definition of each microservice.


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
        external-dns.alpha.kubernetes.io/hostname: api.example.com # Optional
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
        external-dns.alpha.kubernetes.io/hostname: app.example.com # Optional
    spec:
      type: NodePort
      selector:
        app: frontend
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

Observe that we have provided the ExternalDNS configuration for each of the microservices using the `external-dns.alpha.kubernetes.io/hostname` annotation.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- backend.yml
│   |-- frontend.yml
│   |-- ingress.yml
```

Let's apply the manifests to create the kubernetes objects:

```
kubectl apply -f manifests/
```

This will create the following resources:

- Deployment and service for `backend` microservice.
- Deployment and service for `frontend` microservice.
- Ingress with two rules.


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


## Step 3: Verify AWS Resources in AWS Console

Visit the AWS console and verify the resources created by AWS Load Balancer Controller.

Also, go to AWS Route 53 and verify the records (`api.example.com` and `app.example.com`) that were added by ExternalDNS.

You can also check the events that external-dns pod performs:

```
kubectl logs -f <external-dns-pod> -n external-dns
```


## Step 4: Access App Using Route 53 DNS

Once the load balancer is in `Active` state, you can hit the subdomains you created in Route 53 and verify if everything is working properly.

Try accessing the following hosts:

```
# Backend
http://api.example.com

# Frontend
http://app.example.com
```

!!! note
    For this demo, we have not enabled SSL to maintain the focus on the ExternalDNS annotation. However, you can add SSL-specific annotations to enable SSL if needed.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- backend.yml
│   |-- frontend.yml
│   |-- ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

The Route 53 records will also be deleted when the ingress or service is deleted.


<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[reyanshkharga/reactapp:v1]: https://hub.docker.com/r/reyanshkharga/reactapp

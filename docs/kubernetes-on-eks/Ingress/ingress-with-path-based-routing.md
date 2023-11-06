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
    - [reyanshkharga/reactapp:profile]{:target="_blank"} is a react app that serves request on `/profile` path.
    - [reyanshkharga/reactapp:notifications]{:target="_blank"} is a react app that serves request on `/notifications` path.
    - [reyanshkharga/reactapp:feed]{:target="_blank"} is a react app that serves request on `/feed` path.


## Objective

In this example we will have 3 microservices:

1. profile (uses docker image `reyanshkharga/reactapp:profile`)
2. notifications (uses docker image `reyanshkharga/reactapp:notifications`)
3. feed (uses docker image `reyanshkharga/reactapp:feed`)

We'll do the following:

1. Create a deployment and service for `profile` microservice
2. Create a deployment and service for `notifications` microservice
3. Create a deployment and service for `feed` microservice
4. Create a ingress that sends traffic to one of the microservices based on the path
5. We'll also provide separate health check path for each microservie using `alb.ingress.kubernetes.io/healthcheck-path` annotation in the service definition of each microservice



## Create Kubernetes Objects

Let's create the kubernetes objects as discussed above:




<!-- Hyperlinks -->
[reyanshkharga/reactapp:profile]: https://hub.docker.com/r/reyanshkharga/reactapp
[reyanshkharga/reactapp:notifications]: https://hub.docker.com/r/reyanshkharga/reactapp
[reyanshkharga/reactapp:feed]: https://hub.docker.com/r/reyanshkharga/reactapp
---
description: Effortlessly manage traffic and enhance your Kubernetes deployment. Learn how to create Ingress with a Default Backend using our step-by-step guide. Optimize your containerized applications and ensure seamless routing even when no specific rule matches. Get started with Ingress and Default Backend configuration.
---

# Create Ingress With Default Backend

A `defaultBackend` is often configured in an Ingress controller to service any requests that do not match a path in the `spec`.

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

Apply the manifest to create the Deployment:

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


## Step 2: Create a NodePort Service

Let's create a NodePort service as follows:

=== ":octicons-file-code-16: `my-nodeport-service.yml`"

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

Apply the manifest to create the NodePort service:

```
kubectl apply -f my-nodeport-service.yml
```

Verify service:

```
kubectl get svc
```


## Step 3: Create Ingress

Now that we have the service ready, let's create an Ingress object:

=== ":octicons-file-code-16: `my-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing # Default value is internal
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
    spec:
      ingressClassName: alb
      defaultBackend:
        service:
          name: my-nodeport-service
          port:
            number: 5000
      rules:
      - http:
          paths:
          - path: /yXnKHUoJGt
            pathType: Prefix
            backend:
              service:
                name: my-nodeport-service
                port:
                  number: 5000
    ```

Observe the following:

1. We have used annotations to specify load balancer and target group attributes
2. We have one rule that matches `/yXnKHUoJGt` path and then routes traffic to default backend which in this case is `my-nodeport-service`.

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

Pay close attention to the listener rules. You will observe a default listener rule that was set using `defaultBackend`.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Troubleshooting

If you don't see the load balancer in the AWS console, this means the ingress has some issue. To identify the underlying issue, you can examine the logs of the controller as follows:

```
# Describe the ingress
kubectl describe ing my-ingress

# View aws load balancer controller logs
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-nodeport-service.yml
│   |-- my-ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


!!! quote "References:"
    !!! quote ""
        * [Default Backend in Ingress]{:target="_blank"}
        * [Path Types in Ingress]{:target="_blank"}


<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[Default Backend in Ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/#default-backend
[Path Types in Ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

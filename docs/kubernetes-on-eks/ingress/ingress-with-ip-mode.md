---
description: Maximize Kubernetes Ingress capabilities with our comprehensive guide on creating Ingress in IP Mode. Take control of IP-based traffic management for your containerized applications. Learn how to create Ingress with IP Mode and optimize your Kubernetes infrastructure
---

# Create Ingress With IP Mode

You can use <a>`alb.ingress.kubernetes.io/target-type`</a> annotation in the Ingress object to specify how to route traffic to pods. You can choose between `instance` and `ip`.

The default value for `alb.ingress.kubernetes.io/target-type` is `instance`. So, you must define this explicitly if you want to use `ip` mode.


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

The kubernetes service can be `NodePort` or `ClusterIP` to use `ip` mode. So, let's create a `ClusterIP` service since it is more secure:

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

Apply the manifest to create the service:

```
kubectl apply -f my-service.yml
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
        alb.ingress.kubernetes.io/tags: Project=eks-masterclass,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
        alb.ingress.kubernetes.io/target-type: ip # Default is instance
    spec:
      ingressClassName: alb
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
    ```

Observe the following:

1. We have used annotations to specify load balancer and target group attributes
2. We have one rule that matches `/` path and then routes traffic to `my-service`

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

Pay close attention to the `Listeners`, `Rules` and `TargetGroups`.

You will observe that in the Target Group, IPs are registered as targets because we chose `ip` as target type. These IPs are associated with pods that `my-service` is configured to serve the traffic from.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 5: Access App Via Load Balancer DNS

Once the load balancer is in `Active` state, you can hit the load balancer DNS and verify if everything is working properly.

Access the load balancer DNS by entering it in your browser. You can obtain the load balancer DNS either from the AWS console or the Ingress configuration.

Try accessing the following paths:

```
# Root path
<load-balancer-dns>/

# Health path
<load-balancer-dns>/health

# Random generator path
<load-balancer-dns>/random
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
│   |-- my-service.yml
│   |-- my-ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
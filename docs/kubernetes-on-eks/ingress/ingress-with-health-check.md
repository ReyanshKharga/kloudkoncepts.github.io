---
description: Boost application reliability and performance. Explore our guide on creating Kubernetes Ingress with Health Check. Discover how to configure proactive health checks for your services and streamline traffic management. Elevate your Kubernetes deployment with Ingress and Health Check configuration.
---

# Create Ingress With Health Check

Health check on target groups can be controlled with following annotations:

| Annotation | Function |
|------------|------------|
| <a>`alb.ingress.kubernetes.io/healthcheck-protocol`</a> |specifies the protocol used when performing health check on targets. |
| <a>`alb.ingress.kubernetes.io/healthcheck-port`</a> | specifies the port used when performing health check on targets. When using `target-type: instance` with a service of type `NodePort`, the healthcheck port can be set to `traffic-port` to automatically point to the correct port. |
| <a>`alb.ingress.kubernetes.io/healthcheck-path`</a> | specifies the HTTP path when performing health check on targets. |
| <a>`alb.ingress.kubernetes.io/healthcheck-interval-seconds`</a> | specifies the interval(in seconds) between health check of an individual target. |
| <a>`alb.ingress.kubernetes.io/healthcheck-timeout-seconds`</a> | specifies the timeout(in seconds) during which no response from a target means a failed health check. |
| <a>`alb.ingress.kubernetes.io/success-codes`</a> | specifies the HTTP or gRPC status code that should be expected when doing health checks against the specified health check path. |
| <a>`alb.ingress.kubernetes.io/healthy-threshold-count`</a> | specifies the consecutive health checks successes required before considering an unhealthy target healthy. |
| <a>`alb.ingress.kubernetes.io/unhealthy-threshold-count`</a> | specifies the consecutive health check failures required before considering a target unhealthy. |


## Annotation Format

Annotation keys and values can only be strings. Advanced format should be encoded as below:

```yaml
boolean: 'true' # Must be quoted
integer: '42' # Must be quoted
stringList: s1,s2,s3
stringMap: k1=v1,k2=v2
json: 'jsonContent' # Must be quoted
```


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


## Step 2: Create a NodePort Service

Let's create a `NodePort` service as follows:

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

If you don't explicitly provide a `nodePort`, you'll observe that the service is automatically assigned one. However, if desired, you can specify a specific `nodePort`.


## Step 3: Create Ingress

Now that we have the service ready, let's create an Ingress object with health check:

=== ":octicons-file-code-16: `my-ingress.yml`"

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
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-nodeport-service
                port:
                  number: 5000
    ```

Observe the following:

1. We have used annotations to specify load balancer and target group attributes
2. We have one rule that matches `/` path and then routes traffic to `my-nodeport-service`
3. We have specified health check parameters for the target group

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

Go to AWS Console and verify health check configuration of the target group that ingress created.


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
        * [Health Check Annotations]{:target="_blank"}




<!-- Hyperlinks -->
[Health Check Annotations]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/#health-check
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
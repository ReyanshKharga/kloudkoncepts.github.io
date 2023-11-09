---
description: Boost application reliability and performance. Explore our guide on creating Kubernetes Ingress with Health Check. Discover how to configure proactive health checks for your services and streamline traffic management. Elevate your Kubernetes deployment with Ingress and Health Check configuration.
---

# Network Load Balancer With Health Check

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

The Load Balancer Controller currently ignores the `timeout` configuration due to the limitations on the AWS NLB. The default `timeout` for `TCP` is 10s and `HTTP` is 6s.



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


## Step 2: Create a LoadBalancer Service

Let's create a `LoadBalancer` service as follows:

=== ":octicons-file-code-16: `my-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: nlb-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-name: my-nlb
        service.beta.kubernetes.io/aws-load-balancer-type: external
        service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: instance # Must specify this annotation
        service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing # Default is internal
        # Health Check
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-protocol: http
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-port: traffic-port
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-path: /health
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: '10'
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: '2' # ignored
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: '2'
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: '2'
    spec:
      type: LoadBalancer
      selector:
        app: demo
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

Note that we are offloading the reconciliation to AWS Load Balancer Controller using the `service.beta.kubernetes.io/aws-load-balancer-type: external` annotation.


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
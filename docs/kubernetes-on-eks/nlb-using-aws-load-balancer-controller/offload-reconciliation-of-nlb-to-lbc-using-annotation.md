---
description: Easily Offload NLB Reconciliation to AWS Load Balancer Controller with LBC annotation. Simplify management and optimize your Network Load Balancer (NLB) with LBC. 
---

# Offload Reconciliation of NLB to LBC Using Annotation

In this demo, we will use `service.beta.kubernetes.io/aws-load-balancer-type: external` annotation to offload the reconciliation of Network Load Balancer (NLB) to AWS Load Balancer Controller (LBC).

!!! note
    If you configure `service.beta.kubernetes.io/aws-load-balancer-type: external`, you must provide the `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` annotation and set it to either `instance` or `ip`.


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


## Step 2: Create a Service

Now, let's create a `LoadBalancer` service but this time we'll offload the reconciliation of NLB to LBC using `.spec.loadBalancerClass` field.

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
    spec:
      type: LoadBalancer
      selector:
        app: demo
      ports:
        - port: 80
          targetPort: 5000
    ```

!!! note
    The `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` annotation is mandatory when you use the `service.beta.kubernetes.io/aws-load-balancer-type: external` annotation to offload the reconciliation of NLB to LBC.

    Also, to create an internet-facing NLB, the following annotation is required on your service:

    ```yaml
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
    ```


Apply the manifest to create the service:

```
kubectl apply -f my-service.yml
```

Verify service:

```
kubectl get svc
```

## Step 3: Verify the Network Load Balancer (NLB) in AWS Console

Visit AWS Console and verify that a network load balancer (NLB) was created.


Also, verify that the NLB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 4: Access App Via Network Load Balancer DNS

Once the load balancer is in `Active` state, you can hit the load balancer DNS and verify if everything is working properly.

Access the load balancer DNS by entering it in your browser. You can get the load balancer DNS either from the AWS console or the service configuration.

Try accessing the following paths:

```
# Root path
<nlb-dns>/

# Health path
<nlb-dns>/health

# Random generator path
<nlb-dns>/random
```


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
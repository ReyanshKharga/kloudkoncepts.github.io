---
description: Unlock the potential of Kubernetes Ingress in Instance Mode with our step-by-step guide. Create and configure Ingress to efficiently manage traffic routing for your containerized applications.
---

# Create Ingress With Instance Mode

You can use <a>`alb.ingress.kubernetes.io/target-type`</a> annotation in the Ingress object to specify how to route traffic to pods. You can choose between `instance` and `ip`.

The kubernetes service must be of type `NodePort` to use `instance` mode. This is because worker nodes (EC2 instances) are registered as targets in the target group that will be created by the AWS Load Balancer Controller.

The default value for `alb.ingress.kubernetes.io/target-type` is `instance`. So you don't have to define this explicitly unless you want to use `ip` mode.

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


## Step 2: Create a NodePort Service

The kubernetes service must be of type `NodePort` to use `instance` mode. So, let's create a `NodePort` service as follows:

=== ":octicons-file-code-16: `my-nodeport-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeport-service
    spec:
      type: NodePort
      selector:
        app: nodeapp
      ports:
        - port: 80
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
                  number: 80
    ```

Observe the following:

1. We have used annotations to specify load balancer and target group attributes
2. We have one rule that matches `/` path and then routes traffic to `my-nodeport-service`

!!! note
    Before the `IngressClass` resource and `ingressClassName` field were added in Kubernetes 1.18, Ingress classes were specified with a `kubernetes.io/ingress.class` annotation on the Ingress. This annotation was never formally defined, but was widely supported by Ingress controllers.

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

Here's what happens when you create an ingress:

1. An ALB (ELBv2) is created in AWS for the new ingress resource.
2. Target Groups are created in AWS for each unique kubernetes service described in the ingress resource.
3. Listeners are created for every port detailed the ingress resource annotations.
4. Listener rules are created for each path specified in the ingress resource. This ensures traffic to a specific path is routed to the correct kubernetes service.

This is all done by the `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```

You can see the events such as `creating securityGroup`, `created securityGroup`, `creating loadBalancer`, `created loadBalancer`, `created listener`, `created listener rule`, `creating targetGroupBinding`, `created targetGroupBinding`, `successfully deployed model`, etc in the logs.


## Step 4: Verify AWS Resources in AWS Console

Visit the AWS console and verify the resources created by AWS Load Balancer Controller.

Pay close attention to the `Listeners`, `Rules` and `TargetGroups`.

You will observe that in the Target Group, instances are registered as targets because we chose `instance` as target type.

Note that the Load Balancer takes some time to become `Active`.


## Step 5: Access App Via Load Balancer DNS

Once the load balancer is in `Active` state, you can hit the load balancer DNS and verify if everything is working properly.

Access the load balancer DNS by entering it in your browser. You can get the load balancer DNS either from the AWS console or the Ingress configuration.

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
│   |-- my-nodeport-service.yml
│   |-- my-ingress.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
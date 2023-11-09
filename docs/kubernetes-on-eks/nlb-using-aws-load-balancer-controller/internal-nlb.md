---
description:  Effortlessly manage Internal Network Load Balancer (NLB) with AWS Load Balancer Controller. Learn how to configure and optimize internal NLBs seamlessly. Elevate your infrastructure with Internal NLB using Load Balancer Controller.
---

# Configure Internal Network Load Balancer

You can set the value of `service.beta.kubernetes.io/aws-load-balancer-scheme` annotation to `internal` to create an internal network load balancer.

!!! note
    If you have microservices on instances that are registered with a Network Load Balancer, you can't use the load balancer to provide communication between them unless the load balancer is `internet-facing` or the instances are registered by IP address. For more information, see [Connections time out for requests from a target to its load balancer]{:target="_blank"}.

    Therefore, we will use `ip` target type for this example.


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
        service.beta.kubernetes.io/aws-load-balancer-scheme: internal # Default is internal
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


## Step 3: Verify AWS Resources in AWS Console

Visit the AWS console and verify the resources created by AWS Load Balancer Controller.

Pay close attention to the NLB type. It should be internal.

Note that the Load Balancer takes some time to become `Active`.

Also, verify that the NLB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 4: Access App Using Internal Load Balancer DNS

Because the network load balancer is internal, access to our app from outside the VPC is restricted. To overcome this, let's create a pod that we can use to access the load balancer and, in turn, our app. Since the pod will reside within the same VPC, we will be able to access our app.

First, let's create a pod as follows:

=== ":octicons-file-code-16: `nginx-pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
    ```

Apply the manifest to create the pod:

```
kubectl apply -f nginx-pod.yml
```

Now, let's start a shell session inside the nginx container and hit the internal load balancer url:

```
# Start a shell session inside the nginx container
kubectl exec -it nginx -- bash

# Hit the load balancer url using CURL
curl <internal-nlb-dns>
```

You'll see the response from the app.


!!! tip
    Set `target-type` to `instance` and try to access the internal nlb dns from nginx pod. It will succeed for some of the requests while it will fail for others.

    Change the `target-type` to `ip` and try to access the internal nlb dns. It should work fine without any timeouts.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
│   |-- nginx-pod.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


!!! quote "References:"
    !!! quote ""
        * [NLB Timeout Error Reason]{:target="_blank"}
        * [Connections time out for requests from a target to its load balancer]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[Connections time out for requests from a target to its load balancer]: https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-troubleshooting.html#loopback-timeout
[NLB Timeout Error Reason]: https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html
---
description: 
---

# ExternalDNS Demo With Private Hosted Zone

In Amazon Route 53, a private hosted zone is a DNS (Domain Name System) zone that is used for private internal DNS resolution within a Virtual Private Cloud (VPC) or a set of interconnected VPCs. It is separate from a public hosted zone, which is used for DNS resolution on the public internet.

Let's see how ExternalDNS works with private hosted zones.


## Prerequisite

To follow this tutorial, you'll need to create a private hosted zone in the VPC where EKS cluster was created. For example, you can create a private hosted zone called `example.internal`.


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


## Step 2: Create a Service

Next, let's create a service as follows:

=== ":octicons-file-code-16: `my-service.yml`"

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

Apply the manifest to create the service:

```
kubectl apply -f my-service.yml
```

Verify service:

```
kubectl get svc
```


## Step 3: Create Ingress

Now that we have the service ready, let's create an Ingress object with ExternalDNS annotation:

=== ":octicons-file-code-16: `my-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internal # Default value is internal
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
        alb.ingress.kubernetes.io/target-type: instance # Optional
        # external-dns specific configuration for creating route53 record-set
        external-dns.alpha.kubernetes.io/hostname: api.example.internal # give your domain name here (Optional)
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

Observe that we have set `alb.ingress.kubernetes.io/scheme` to `internal` and therefore the AWS Load Balancer Controller will create an internal load balancer. Also, ExternalDNS will create a Route 53 record record `api.example.internal` in the private hosted zone we created because we have set `external-dns.alpha.kubernetes.io/hostname` to `api.example.internal`.

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

Also, go to AWS Route 53 and verify the record (`api.example.internal`) that was added by ExternalDNS.

You can also check the events that external-dns pod performs:

```
kubectl logs -f <external-dns-pod> -n external-dns
```


## Step 5: Access App Using Internal Load Balancer DNS

Because the load balancer is internal, access to our app from outside the VPC is restricted. To overcome this, let's create a pod that we can use to access the load balancer and, in turn, our app. Since the pod will reside within the same VPC, we will be able to access our app.

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

Now, let's start a shell session inside the nginx container and hit the private Route 53 DNS:

```
# Start a shell session inside the nginx container
kubectl exec -it nginx -- bash

# Hit the url using CURL
curl api.example.internal
```

You'll see the response from the app.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
│   |-- my-ingress.yml
│   |-- nginx-pod.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

The Route 53 record will also be deleted when the ingress or service is deleted.



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
---
description: Secure your applications and enhance user trust. Learn how to create Kubernetes Ingress with SSL in our comprehensive guide. Discover the essential steps to enable SSL encryption for your services and protect sensitive data. Elevate your Kubernetes deployment with SSL-enabled Ingress.
---

# Create Ingress With SSL

SSL support can be controlled with the following annotations:

| Annotation | Function |
|----------------------------------------------|----------|
| <a>`alb.ingress.kubernetes.io/certificate-arn`</a> |specifies the `ARN` of one or more certificate managed by AWS Certificate Manager. The first certificate in the list will be added as default certificate. And remaining certificate will be added to the optional certificate list. |
| <a>`alb.ingress.kubernetes.io/ssl-policy`</a> | specifies the Security Policy that should be assigned to the ALB, allowing you to control the protocol and ciphers. This is optional and defaults to `ELBSecurityPolicy-2016-08` |


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
      replicas: 1
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

Now that we have the service ready, let's create an Ingress object with SSL:

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
        # SSL Annotations
        alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:170476043077:certificate/2d88e035-cde7-472a-9cd3-6b6ce6ece961
        alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-2016-08 # Optional
        # Listerner Ports Annotation
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
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

Be sure to replace the value of `alb.ingress.kubernetes.io/certificate-arn` with the `ARN` of the SSL certificate you created.

!!! note
    `alb.ingress.kubernetes.io/listen-ports` defaults to `'[{"HTTP": 80}]'` or `'[{"HTTPS": 443}]'` depending on whether `certificate-arn` is specified.

    If you want to serve both `HTTP` and `HTTPS` traffic, you must set `alb.ingress.kubernetes.io/listen-ports` to `'[{"HTTP": 80}, {"HTTPS": 443}]'`.

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

Pay close attention to the certificate attached to the `HTTPS (443)` listener in the load balancer.

Also, verify that the ALB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```


## Step 5: Add Record in Route 53

Go to AWS Route 53 and add an `A` record (e.g `api.example.com`) for your domain that points to the Load Balancer. You can use alias to point the subdomain to the load balancer that was created.


## Step 6: Access App Using Route 53 DNS

Once the load balancer is in `Active` state, you can hit the subdomain you created in Route 53 and verify if everything is working properly.

Try accessing the following paths:

```
# Root path
https://api.example.com/

# Health path
https://api.example.com/health

# Random generator path
https://api.example.com/random
```

Verify that both `HTTP` and `HTTPS` works.



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

Also, go to Route 53 and delete the `A` record that you created.




!!! quote "References:"
    !!! quote ""
        * [Registering a New Domain in Route 53]{:target="_blank"}
        * [SSL Annotations]{:target="_blank"}
        * [Security Policies]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[Registering a New Domain in Route 53]: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html#domain-register-procedure
[SSL Annotations]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/#ssl
[Security Policies]: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html#describe-ssl-policies

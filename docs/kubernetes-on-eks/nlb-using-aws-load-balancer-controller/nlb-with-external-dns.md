---
description: Unlock the power of Network Load Balancer (NLB) with External DNS. Learn to seamlessly integrate and optimize NLB for public access. Configure External DNS to enhance your NLB capabilities.
---

# Network Load Balancer With External DNS

You can use the `external-dns.alpha.kubernetes.io/hostname`` annotation to let ExternalDNS manage the Route 53 record for the kubernetes service.


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
        # TLS
        service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:ap-south-1:170476043077:certificate/2d88e035-cde7-472a-9cd3-6b6ce6ece961
        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: '443'
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
        #External DNS
        external-dns.alpha.kubernetes.io/hostname: api.example.com # give your domain name here (Optional)
    spec:
      type: LoadBalancer
      selector:
        app: demo
      ports:
        - port: 443 # Creates a listener with port 443
          targetPort: 5000
    ```

Be sure to replace the value of `alb.ingress.kubernetes.io/certificate-arn` with the `ARN` of the SSL certificate you created.

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

Pay close attention to the health check configuration of the target group.

Note that the Load Balancer takes some time to become `Active`.

Verify that the NLB was created by `AWS Load Balancer Controller`. You can check the events in the logs as follows:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller --all-containers=true
```

Also, go to AWS Route 53 and verify the record (`api.example.com`) that was added by ExternalDNS.

You can also check the events that external-dns pod performs:

```
kubectl logs -f <external-dns-pod> -n external-dns
```


## Step 4: Access App Using Route 53 DNS

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
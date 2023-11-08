---
description: See ExternalDNS in action. Watch our demo of ExternalDNS with a single host. Learn how this tool helps you make Kubernetes resources discoverable with public DNS servers.
---

# ExternalDNS Demo With One Host

Simply add the `external-dns.alpha.kubernetes.io/hostname` annotation to either the kubernetes Ingress or Service, and ExternalDNS will use this information to create corresponding Route 53 records.

Let's see this in action!


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
        alb.ingress.kubernetes.io/scheme: internet-facing # Default value is internal
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=DevOps # Optional
        alb.ingress.kubernetes.io/load-balancer-name: my-load-balancer # Optional
        alb.ingress.kubernetes.io/target-type: instance # Optional
        # external-dns specific configuration for creating route53 record-set
        external-dns.alpha.kubernetes.io/hostname: api.example.com # give your domain name here (Optional)
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

Be sure to replace the value of `external-dns.alpha.kubernetes.io/hostname` with your domain.

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

Also, go to AWS Route 53 and verify the record (`api.example.com`) that was added by ExternalDNS.

You can also check the events that external-dns pod performs:

```
kubectl logs -f <external-dns-pod> -n external-dns
```


## Step 5: Access App Using Route 53 DNS

Once the load balancer is in `Active` state, you can hit the subdomain you created in Route 53 and verify if everything is working properly.

Try accessing the following paths:

```
# Root path
http://api.example.com/

# Health path
http://api.example.com/health

# Random generator path
http://api.example.com/random
```

!!! note
    For this demo, we have not enabled SSL to maintain the focus on the ExternalDNS annotation. However, you can add SSL-specific annotations to enable SSL if needed.


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

The Route 53 record will also be deleted when the ingress or service is deleted.



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
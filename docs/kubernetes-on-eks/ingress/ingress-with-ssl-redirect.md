---
description: Ensure data security and a seamless user experience. Explore our guide on creating Kubernetes Ingress with SSL Redirect. Learn how to set up SSL redirection to enforce secure connections for your services. Elevate your application's security and user satisfaction with SSL Redirect-enabled Ingress.
---

# Create Ingress With SSL Redirect

The <a>`alb.ingress.kubernetes.io/ssl-redirect`</a> annotation enables SSLRedirect and specifies the SSL port that redirects to.


## Prerequisite

To follow this tutorial, you'll require a domain and, additionally, an SSL certificate for the domain and its subdomains.

1. Register a Route53 Domain

    Go to AWS Console and register a Route53 domain. You can opt for a cheaper TLD (top level domain) such as `.link`

    !!! note
        It usually takes about 10 minutes but it might take about an hour for the registered domain to become available.

2. Request a Public Certificate

    Visit AWS Certificate Manager in AWS Console and request a public certificate for your domain and all the subdomains. For example, if you registered for a domain `example.com` then request certificate for `example.com` and `*.example.com`

    !!! note
        Make sure you request the certificate in the region where your EKS cluster is in.

3. Validate the Certificate

    Validate the requested certificate by adding `CNAME` records in Route53. It is a very simple process. Go to the certificate you created and click on `Create records in Route53`. The `CNAMEs` will be automatically added to Route53.

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

Now that we have the service ready, let's create an Ingress object with SSL redirect:

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
        # SSL Redicrect Annotation
        alb.ingress.kubernetes.io/ssl-redirect: '443'
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

Observe that we have added the `alb.ingress.kubernetes.io/ssl-redirect` annotation with value `443`. With this annotation in place, every `HTTP` listener will be configured with a default action which redirects to `HTTPS`, and other rules will be ignored.

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

Pay close attention to the default listener rule and certificate attached to the `HTTPS (443)` listener in the load balancer.

You'll notice that the `HTTP` is being redirected to `HTTPS`.


## Step 5: Add Record in Route53

Go to AWS Route53 and add an `A` record (e.g `api.example.com`) for your domain that points to the Load Balancer. You can use alias to point the subdomain to the load balancer that was created.


## Step 6: Access App Using Route53 DNS

Once the load balancer is in `Active` state, you can hit the subdomain you created in Route53 and verify if everything is working properly.

Try accessing the following paths:

```
# Root path
https://api.example.com/

# Health path
https://api.example.com/health

# Random generator path
https://api.example.com/random
```

Also, verify that `HTTP` is redirected to `HTTPS`.



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

Also, go to Route53 and delete the `A` record that you created.




!!! quote "References:"
    !!! quote ""
        * [SSL Redirect]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp
[SSL Redirect]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/#traffic-listening

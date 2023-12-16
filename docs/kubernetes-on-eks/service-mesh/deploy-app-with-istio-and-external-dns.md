---
description: Learn how to effortlessly deploy applications using Istio and ExternalDNS with our comprehensive guide. Optimize your workflow, streamline deployment, and enhance your application management effortlessly.
---

# Deploy Application With Istio and ExternalDNS

Now that we have updated our ExternalDNS setup, let's deploy a simple application using traffic management API resources such as Gateway and Virtual Service along with ExternalDNS configurations.


## Step 1: View Istio Proxy Configuration

Note down the current proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```

It should look something like this:

```
NAME     VHOST NAME     DOMAINS     MATCH                  VIRTUAL SERVICE
         backend        *           /stats/prometheus*     
         backend        *           /healthz/ready*  
```


## Step 2: Deploy the Application

Prepare the manifest files for our application as follows:

=== ":octicons-file-code-16: `app.yml`"

    ```yaml linenums="1"
    ---         
    apiVersion: v1
    kind: Namespace
    metadata:
      name: test
      labels:
        istio-injection: enabled
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: test-deploy
      namespace: test
      labels:
        app: test-app
        version: v1
    spec: 
      replicas: 1
      selector:
        matchLabels:
          app: test-app
      template:
        metadata:
          labels:
            app: test-app
            version: v1
        spec: 
          containers:
          - name: web
            image: reyanshkharga/nodeapp:v1
            ports:
            - containerPort: 5000
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            readinessProbe:
              httpGet:
                path: /health
                port: 5000
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: test-svc
      namespace: test
    spec:   
      selector:
        app: test-app
      ports:
        - name: http
          protocol: TCP
          port: 80
          targetPort: 5000
    ```

=== ":octicons-file-code-16: `gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: test-gateway
      namespace: test
    spec: 
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "test.example.com"
    ```

=== ":octicons-file-code-16: `virtualservice.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: test-virtualservice
      namespace: test
      # We need to add this annotation for ExternalDNS to automatically add the record to Route53
      # The current ExternalDNS implementation for istio doesn't support automatic picking of load balancer dns
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "test.example.com"
      gateways:
      - test-gateway
      http:
      - match: 
        - uri:   
            prefix: /
        route:
        - destination:
            host: test-svc
            port:
              number: 80
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` with the load balancer DNS that was created by ingress we created for Istio.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- app.yml
│   |-- gateway.yml
│   |-- virtualservice.yml
```

Apply the manifests to deploy the application with istio gateway and virtual service:

```
kubectl apply -f manifests/
```

This will create the following kubernetes and Istio resources:

- Kubernetes namespace
- Kubernetes Deployment
- Kubernetes Service
- Istio Gateway
- Istio Virtual Service

!!! note
    The current implementation of ExternalDNS for istio is not fully mature. The current ExternalDNS implementation for istio doesn't support automatic picking of load balancer DNS.

    We need to provide the application load balancer DNS for ExternalDNS to automatically add the record to Route53. We can do so using the <a>`external-dns.alpha.kubernetes.io/target`</a> annotation.


## Step 3: View the Updated Istio Proxy Configuration

View the updated proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```

It should looks something like this:

```
NAME        VHOST NAME            DOMAINS            MATCH                 VIRTUAL SERVICE
http.8080   test.example.com:80   test.example.com   /*                    test-virtualservice.test
            backend               *                  /stats/prometheus*     
            backend               *                  /healthz/ready*  
```


## Step 4: Verify DNS Record in Route 53

Go to AWS Route 53 and verify if a DNS record that points `test.example.com` to the load balancer was created.

Open any browser and visit `test.example.com` to verify app.

Here's how the traffic flows across Istio and kubernetes resources:

``` mermaid
graph LR
  A(User) --> B("Application Load Balancer
  (associated with istio ingress gateway)");
  B --> C("test-gateway
  (Istio Gateway)");
  C --> D("test-virtualservice
  (Istio Virtual Service)");
  D --> E(test-svc);
```


## Clean Up

Delete the kubernetes and istio resources we created:

```
kubectl delete -f manifests/
```
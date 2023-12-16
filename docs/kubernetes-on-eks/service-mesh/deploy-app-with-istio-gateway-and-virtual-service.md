---
description: Learn how to deploy your application effortlessly using Istio Gateway and Virtual Service. Explore simple steps for leveraging traffic management API resources to enhance your application deployment with Istio.
---

# Deploy Application With Istio Gateway and Virtual Service

Now that we have a good understanding of traffic management in Istio, let's deploy a simple application using traffic management API resources such as Gateway and Virtual Service.


## Step 1: View Istio Proxy Configuration

Note down the current proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
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


## Step 3: View the Updated Istio Proxy Configuration

View the updated proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```


## Step 4: Add DNS Record in Route 53

Go to AWS Route 53 and add a record that points `test.example.com` to the load balancer created by Istio Ingress Gateway.

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

In the next section we will update `ExternalDNS` to make it work with Istio so that DNS records are automatically created in Route 53 based on hosts provided in Istio virtual services.


## Clean Up

Delete the kubernetes and istio resources we created:

```
kubectl delete -f manifests/
```
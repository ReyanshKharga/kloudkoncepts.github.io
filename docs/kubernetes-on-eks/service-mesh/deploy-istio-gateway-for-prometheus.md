---
description: Learn how to efficiently set up Istio Gateway for Prometheus. Elevate your system observability and performance management effortlessly.
---

# Deploy Istio Gateway for Prometheus

Recall that we deleted the ingress we created for Prometheus. Now, let's enable external access for Prometheus using the Istio Gateway.


## Step 1: View Istio Proxy Configuration

Note down the current proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```


## Step 2: Deploy Istio Gateway for Prometheus

Prepare istio manifests for Prometheus as follows:

=== ":octicons-file-code-16: `prometheus-gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: prometheus-gateway
      namespace: prometheus
    spec: 
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "prometheus.example.com"
    ```

=== ":octicons-file-code-16: `prometheus-virtualservice.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: prometheus-virtualservice
      namespace: prometheus
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "prometheus.example.com"
      gateways:
      - prometheus-gateway
      http:
      - match: 
        - uri:   
            prefix: /
        route:
        - destination:
            host: prometheus-server
            port:
              number: 80
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` with the load balancer DNS that was created by ingress we created for Istio.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- prometheus-gateway.yml
│   |-- prometheus-virtualservice.yml
```

Apply the manifests to deploy the istio gateway for prometheus:

```
kubectl apply -f manifests/
```


## Step 3: View the Updated Istio Proxy Configuration

View the updated proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```


## Step 4: Verify DNS Record in Route 53

Go to AWS Route 53 and verify whether a DNS record was created for the host we defined in the virtual service, pointing to the Istio load balancer.



## Step 5: Access Prometheus

Open any browser on your local host machine and hit the Prometheus host URL to access prometheus:

```
https://prometheus.example.com
```
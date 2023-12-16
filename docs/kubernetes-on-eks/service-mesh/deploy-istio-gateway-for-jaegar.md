---
description: Learn how to efficiently set up Istio Gateway for Jaegar. Elevate your system observability and performance management effortlessly.
---

# Deploy Istio Gateway for Jaegar

Recall that we deleted the ingress we created for Jaegar. Now, let's enable external access for Jaegar using the Istio Gateway.


## Step 1: View Istio Proxy Configuration

Note down the current proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```


## Step 2: Deploy Istio Gateway for Jaegar

Prepare istio manifests for Jaegar as follows:

=== ":octicons-file-code-16: `jaegar-gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: jaegar-gateway
      namespace: istio-system
    spec: 
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "jaegar.example.com"
    ```

=== ":octicons-file-code-16: `jaegar-virtualservice.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: jaegar-virtualservice
      namespace: istio-system
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "jaegar.example.com"
      gateways:
      - jaegar-gateway
      http:
      - match: 
        - uri:   
            prefix: /
        route:
        - destination:
            host: tracing
            port:
              number: 80
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` with the load balancer DNS that was created by ingress we created for Istio.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- jaegar-gateway.yml
│   |-- jaegar-virtualservice.yml
```

Apply the manifests to deploy the istio gateway for jaegar:

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



## Step 5: Access Jaegar

Open any browser on your local host machine and hit the Jaegar host URL to access jaegar:

```
https://jaegar.example.com
```
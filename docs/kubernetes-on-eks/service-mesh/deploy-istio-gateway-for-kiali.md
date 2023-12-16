---
description: Learn how to efficiently set up Istio Gateway for Kiali. Elevate your system observability and performance management effortlessly.
---

# Deploy Istio Gateway for Kiali

Recall that we deleted the ingress we created for Kiali. Now, let's enable external access for Kiali using the Istio Gateway.


## Step 1: View Istio Proxy Configuration

Note down the current proxy configuration:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```


## Step 2: Deploy Istio Gateway for Kiali

Prepare istio manifests for Kiali as follows:

=== ":octicons-file-code-16: `kiali-gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: kiali-gateway
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
        - "kiali.example.com"
    ```

=== ":octicons-file-code-16: `kiali-virtualservice.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: kiali-virtualservice
      namespace: istio-system
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "kiali.example.com"
      gateways:
      - kiali-gateway
      http:
      - match: 
        - uri:   
            prefix: /
        route:
        - destination:
            host: kiali
            port:
              number: 20001
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` with the load balancer DNS that was created by ingress we created for Istio.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- kiali-gateway.yml
│   |-- kiali-virtualservice.yml
```

Apply the manifests to deploy the istio gateway for kiali:

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



## Step 5: Access Kiali

Open any browser on your local host machine and hit the Kiali host URL to access kiali:

```
https://kiali.example.com
```
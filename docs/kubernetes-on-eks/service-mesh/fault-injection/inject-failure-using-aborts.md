---
description: Discover the power of failure injection through deliberate aborts! Learn how to strategically introduce delays to enhance system resilience, uncover weaknesses, and optimize performance.
---

# Inject Failure Using Aborts

Let's inject the fault for certain percentage of the traffic. We will send a fixed response with HTTP code `503`.


## Step 1: Deploy Application

First, let's deploy the application and other Istio components:

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: fault-injection
      labels:
        istio-injection: enabled
    ```

=== ":octicons-file-code-16: `nginx-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      namespace: fault-injection
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
          version: v1
      template:
        metadata:
          labels:
            app: nginx
            version: v1
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v1
            imagePullPolicy: Always
    ```

=== ":octicons-file-code-16: `nginx-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
      namespace: fault-injection
    spec:
      type: ClusterIP
      selector:
        app: nginx
      ports:
        - port: 80
          targetPort: 80
          name: http
    ```

=== ":octicons-file-code-16: `gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: nginx-gateway
      namespace: fault-injection
    spec: 
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "nginx-app.example.com"
    ```

=== ":octicons-file-code-16: `virtual-service.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: nginx-virtualservice
      namespace: fault-injection
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "nginx-app.example.com"
      gateways:
      - nginx-gateway
      http:
      - route:
        - destination:
            host: nginx-service
        fault:
          abort:
            percentage:
              value: 80
            httpStatus: 503
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` annotation in virtual service with the istio load balancer DNS.

Observe that we are injecting abort failure with HTTP status code 503 for 80% of the traffic.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nginx-deployment.yml
│   |-- nginx-service.yml
│   |-- gateway.yml
│   |-- virtual-service.yml
```

Let's apply the manifests to create the kubernetes and istio objects:

```
kubectl apply -f manifests/
```

Verify if the istio proxies are created for the application:

```
# Retrieve proxy configuration
istioctl proxy-config routes svc/istio-ingressgateway -n istio-system
```

Open the application in any browser and see if you receive `503` error for some of the requests.


## Step 2: Generate Load and Verify Failure Percentage in Kiali

Let's use a script to automate traffic generation and record the failure rate:

=== ":octicons-file-code-16: `generate-traffic.sh`"

    ```bash linenums="1"
    #!/bin/bash
    while true
    do
      curl -s -f -o /dev/null "https://nginx-app.example.com"
      echo "sleeping for 0.5 seconds"
      sleep 0.5
    done
    ```

Now, let's execute the script to generate traffic:

```
# Give execute permission to script
chmod +x generate-traffic.sh

# Execute script
./generate-traffic.sh
```

Wait for about 5 minutes and then view error rate in kiali. You'll notice that about 80% of the traffic fails with `503` HTTP error code.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nginx-deployment.yml
│   |-- nginx-service.yml
│   |-- gateway.yml
│   |-- virtual-service.yml
```

Let's delete all the kubernetes and istio resources we created:

```
kubectl delete -f manifests/
```
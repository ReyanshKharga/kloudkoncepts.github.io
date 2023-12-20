---
description: Discover the power of failure injection through deliberate delays! Learn how to strategically introduce delays to enhance system resilience, uncover weaknesses, and optimize performance.
---

# Inject Failure Using Delays

Let's inject failure using delays for 100% of the traffic.


## Step 1: Deploy Application

First, let's deploy the application and other Istio components:

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: delay-injection
      labels:
        istio-injection: enabled
    ```

=== ":octicons-file-code-16: `nginx-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      namespace: delay-injection
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
      namespace: delay-injection
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
      namespace: delay-injection
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
      namespace: delay-injection
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
          delay:
            percentage:
              value: 100
            fixedDelay: 5s
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` annotation in virtual service with the istio load balancer DNS.

Observe that we are injecting a delay of 5 seconds for 100% of the traffic.

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

Open the application in any browser and see if there is actually a delay in the response. Now, failure depends on the application, i.e., the application can set a timeout for the response and send a failure response if timed out.

In our case, we have not implemented a timeout in our application, so you would only notice the delay and not the actual failure due to a timeout.



## Step 2: Generate Load and Verify Delay in Response

Let's use a script to automate traffic generation and record the delay:

=== ":octicons-file-code-16: `generate-traffic.sh`"

    ```bash linenums="1"
    #!/bin/bash
    while true
    do 
        echo $(curl https://nginx-app.example.com/ --silent --output /dev/null -w %{time_total}) seconds
    done
    ```

Now, let's execute the script to generate traffic:

```
# Give execute permission to script
chmod +x generate-traffic.sh

# Execute script
./generate-traffic.sh
```

Observe the response time of each requests.

!!! note
    You won't see the delay in Kiali because Kiali displays latency, not delay. Latency refers to the time from when a request reaches the server until the response is returned, whereas delay is the time from when a request is initiated to when the response is served.



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
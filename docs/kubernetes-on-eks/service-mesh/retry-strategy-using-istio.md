---
description: Explore effective retry strategies with Istio to enhance reliability and performance. Learn how to optimize retries for smoother operations.
---

# Retry Strategy Using Istio

A retry setting specifies the maximum number of times an envoy proxy attempts to connect to a service if the initial call fails. Retries can enhance service availability and application performance by making sure that calls don’t fail permanently because of transient problems such as a temporarily overloaded service or network.

Retry strategy is implemented using `VirtualService`.

Also, note that the default retry behavior for HTTP requests is to retry twice before returning the error. But you can customize this as we will see later in this section.



## Step 1: Deploy Application

First, let's deploy the application and other Istio components:

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: retry-strategy
      labels:
        istio-injection: enabled
    ```

=== ":octicons-file-code-16: `nodeapp-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nodeapp-deployment
      namespace: retry-strategy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nodeapp
          version: v1
      template:
        metadata:
          labels:
            app: nodeapp
            version: v1
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:buggy
            imagePullPolicy: Always
            env:
            - name: BUGGY
              value: "503"
    ```

=== ":octicons-file-code-16: `nodeapp-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: nodeapp-service
      namespace: retry-strategy
    spec:
      type: ClusterIP
      selector:
        app: nodeapp
      ports:
        - port: 80
          targetPort: 5000
          name: http
    ```

=== ":octicons-file-code-16: `gateway.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: nodeapp-gateway
      namespace: retry-strategy
    spec: 
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "nodeapp.example.com"
    ```

=== ":octicons-file-code-16: `virtual-service.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: nodeapp-virtualservice
      namespace: retry-strategy
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "nodeapp.example.com"
      gateways:
      - nodeapp-gateway
      http:
      - route:
        - destination:
            host: nodeapp-service
        retries:
          attempts: 2
          perTryTimeout: 2s
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` annotation in virtual service with the istio load balancer DNS.


!!! note
    The application uses `reyanshkharga/nodeapp:buggy` image with an environment variable `BUGGY` and the value is set to `503`. With this configuration, the app returns `503` error intermittently. More precisely, it generates a random number between 1 and 10 and returns `503` if the random number is 9.


Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nodeapp-deployment.yml
│   |-- nodeapp-service.yml
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



## Step 2: Generate Traffic Using Fortio

Deploy a fortio pod as follows:

=== ":octicons-file-code-16: `fortio-traffic-generator.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name:  traffic-generator
      namespace: circuit-breaker
      labels:
        app: traffic-generator
        version: v1
    spec:
      containers:
      - name: fortio
        image: fortio/fortio
    ```

```
kubectl apply -f fortio-traffic-generator.yml
```

Now, let's generate traffic on our nodeapp service using fortio.

Note that since the retry is implemented using VirtualService meaning the traffic routing rules apply when a host is addressed as defined by the VirtualService.

So, you need to address the host and not the service when testing using fortio.

```
# Set fortio pod name
export FORTIO_POD=traffic-generator

# Call nodeapp service using fortio pod
kubectl exec $FORTIO_POD -n retry-strategy -c fortio -- /usr/bin/fortio curl -quiet https://nodeapp.example.com

# Generate load on nodeapp service using fortio
kubectl exec $FORTIO_POD -n retry-strategy -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning -quiet https://nodeapp.example.com
```

!!! note
    The following command sends a total of 30 HTTP requests to the host with 3 concurrent connections. The `qps` configures queries per second; setting it to 0 indicates sending requests as fast as possible without a specific limit.

    ```
    kubectl exec $FORTIO_POD -n retry-strategy -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning -quiet https://nodeapp.example.com
    ```

You'll notice almost all the requests get a successful response (200) although the service is degraded (intermittent 503).

You can check kiali and actually see that the service is degraded (5xx error) but on the client end you get successfull response due to retries.

You can customize the retry config using VirtualService depending on your your application and use case.




## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nodeapp-deployment.yml
│   |-- nodeapp-service.yml
│   |-- gateway.yml
│   |-- virtual-service.yml
│   |-- fortio-traffic-generator.yml
```

Let's delete all the kubernetes and istio resources we created:

```
kubectl delete -f manifests/
```
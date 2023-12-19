---
description: Learn how to effectively manage web traffic using Istio with our comprehensive guide. Explore traffic splitting techniques to optimize performance and enhance user experience.
---

# Traffic Splitting Using Istio

Traffic splitting helps compare different versions of something (like a webpage or marketing campaign) to find the best-performing option, leading to better decisions, improved user experiences, and reduced risks before implementing changes widely.

Suppose you have two versions of an application. Let's say `v1` and `v2`. You want to send only some traffic to `v2` before completely switching to `v2`. How do you do that?

Istio allows you to split traffic between different versions of an application.

For that you first create a istio `DestinationRule` that defines the subsets of applications you have (`v1` and `v2` in this case).

Once you have defined the subsets, you can configure istio `VirtualService` to split traffic between the subsets. You can mention the weight for each subsets. The weight is the percentage of traffic the subset receives.

For example, imagine you have a service called `reviews`. Currently, it's running version `v1`. Now, you've made some updates (let's call this version `v2`). Initially, you only want 20% of the traffic to go to the new `v2` version of the service, while the rest continues to use the original `v1` version.

``` mermaid
graph LR
  A(Traffic) --> G(Gateway);
  G --> B(Virtual Service);
  B -->|80%| C("Destination Rule
  (subeset v1)");
  B -->|20%| D("Destination Rule
  (subeset v2)");
  C --> E(Reviews - v1);
  D --> F(Reviews - v2);
```


## Step 1: Deploy Application With Multiple Versions

First, let's deploy two versions of the application: v1 and v2, along with other Istio components like Destination Rule, Virtual Service, and Gateway.

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: traffic-split
      labels:
        istio-injection: enabled
    ```

=== ":octicons-file-code-16: `nginx-deployment-v1.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment-v1
      namespace: traffic-split
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

=== ":octicons-file-code-16: `nginx-deployment-v2.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment-v2
      namespace: traffic-split
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
          version: v2
      template:
        metadata:
          labels:
            app: nginx
            version: v2
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v2
            imagePullPolicy: Always
    ```

=== ":octicons-file-code-16: `nginx-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
      namespace: traffic-split
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
      namespace: traffic-split
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

=== ":octicons-file-code-16: `destination-rule.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: nginx-destinationrule
      namespace: traffic-split
    spec:
      host: nginx-service
      subsets:
        - name: v1
          labels:
            version: "v1"
        - name: v2
          labels:
            version: "v2"
    ```

=== ":octicons-file-code-16: `virtual-service.yml`"

    ```yaml linenums="1"
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: nginx-virtualservice
      namespace: traffic-split
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
            subset: v1
          weight: 80
        - destination:
            host: nginx-service
            subset: v2
          weight: 20
    ```

Make sure to replace the value of `external-dns.alpha.kubernetes.io/target` annotation in virtual service with the istio load balancer DNS.

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nginx-deployment-v1.yml
│   |-- nginx-deployment-v2.yml
│   |-- nginx-service.yml
│   |-- gateway.yml
│   |-- destination-rule.yml
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


## Step 2: Generate Load and Verify Traffic Distribution in Kiali

Let's generate some traffic and verify the traffic distribution in kiali.

First, let's write a script that generates traffic:

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

Wait for about 5 minutes and then view trafic distribution graph in kiali. Verify that the traffic distribution closely matches the defined weights for each subsets: 80% to `v1` and 20% to `v2`.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- 00-namespace.yml
│   |-- nginx-deployment-v1.yml
│   |-- nginx-deployment-v2.yml
│   |-- nginx-service.yml
│   |-- gateway.yml
│   |-- destination-rule.yml
│   |-- virtual-service.yml
```

Let's delete all the kubernetes and istio resources we created:

```
kubectl delete -f manifests/
```
---
description: Discover seamless Canary Deployment techniques with Istio. Learn how to split traffic between application versions effectively for optimal performance and user experience. 
---

# Canary Deployment Using Istio


Canary deployment is a gradual release strategy in software development. It involves rolling out new application versions to a small subset of users before a full release, allowing developers to monitor performance and address issues before reaching the entire user base.

Suppose you have two versions of an application. Let's say `v1` and `v2`. You may want to serve version `v2` of the application only to a subset of users (like those with the cookie `version=v2` set in the header — often injected by developers for particular users). How do you do that?

Istio makes canary deployment simple and straightforward.

For that you first create a istio `DestinationRule` that defines the subsets of applications you have (`v1` and `v2` in this case).

Once you have defined the subsets, you can configure Istio `VirtualService` to split traffic between them based on matching conditions. For instance, if the cookie `version=v2` is found in the header, you can route the traffic to `v2`; otherwise, direct it to `v1`.


``` mermaid
graph LR
  A(Traffic) --> G(Gateway);
  G --> B(Virtual Service);
  B -->|"If cookie 'version=v2' 
  is set"| C("Destination Rule
  (subeset v2)");
  B -->|"Else"| D("Destination Rule
  (subeset v1)");
  C --> E(Reviews - v2);
  D --> F(Reviews - v1);
```


## Step 1: Deploy Application With Multiple Versions

First, let's deploy two versions of the application: v1 and v2, along with other Istio components like Destination Rule, Virtual Service, and Gateway.

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: canary-deployment
      labels:
        istio-injection: enabled
    ```

=== ":octicons-file-code-16: `nginx-deployment-v1.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment-v1
      namespace: canary-deployment
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
      namespace: canary-deployment
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
      namespace: canary-deployment
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
      namespace: canary-deployment
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
      namespace: canary-deployment
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
      namespace: canary-deployment
      annotations:
        external-dns.alpha.kubernetes.io/target: "istio-load-balancer-1556246780.ap-south-1.elb.amazonaws.com"
    spec: 
      hosts:
      - "nginx-app.example.com"
      gateways:
      - nginx-gateway
      http:
      - match:
        - headers:
            cookie:
              regex: "^(.*?;)?(version=v2)(;.*)?$"
        route:
        - destination:
            host: nginx-service
            subset: v2
      - route:
        - destination:
            host: nginx-service
            subset: v1
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


## Step 2: Verify if Cookie is Working as Expected

Usually developers can inject cookies for specific users but in this case we'll do it manually via the browser.

hit the host url in the browser and then do the following:

Go to inspect -> application -> cookies (expand it) -> click on your site url -> add a cookie

```
# Add cookie
version=v2
```

Now open a new tab and hit the host url again. You'll see that only the `v2` version of the nginx app is served.


## Step 3: Generate Load and Verify Traffic Distribution in Kiali

First hit the host url without any cookie and view the traffic distrubution in kiali. You'll see that 100% of the traffic is sent to `v1` of the nginx app.

After some time also hit the url with cookies and view the traffic distribution in kiali. With correct cookie, you will see that the traffic is also sent to `v2` version of the app.

Let's also use a script to generate some traffic and verify the traffic distribution in kiali.

First, let's write a script that generates traffic:

=== ":octicons-file-code-16: `generate-traffic.sh`"

    ```bash linenums="1"
    #!/bin/bash
    while true
    do
      curl -s -f -o /dev/null "https://nginx-app.example.com"
      echo "sleeping for 0.5 seconds"
      sleep 0.5
      curl -s -f -o /dev/null "https://nginx-app.example.com" --cookie "version=v2"
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

Wait for about 5 minutes and then view trafic distribution graph in kiali. Verify that the traffic is equally distributed between `v1` and `v2` versions of the app.


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
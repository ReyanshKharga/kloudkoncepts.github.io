---
description: Discover how to enhance your Istio installation by configuring it with the powerful AWS Application Load Balancer. Explore the step-by-step process of setting up Istio with this advanced load balancer for optimized performance and superior features.
---

# Install Istio With Application Load Balancer

By default, Istio creates an AWS Classic Load Balancer during installation. However, we'll be setting up Istio to utilize the AWS Application Load Balancer due to its superior features compared to the Classic Load Balancer.


## Step 1: Install Istio with NodePort Service Type

By default, Istio creates an AWS Classic Load Balancer during installation because the `istio-ingressgateway` service type is set to `LoadBalancer`.

We'll change the service type to `NodePort`, so no load balancer is created at first. Then, we'll create an ingress object for the `istio-ingressgateway` service, which will create an AWS Application Load Balancer using the AWS Load Balancer Controller.

Install Istio with service type set to NodePort:

```
# Install istio
istioctl install --set profile=default --set values.gateways.istio-ingressgateway.type=NodePort -y
```

The output should look similar to the below:

```
✔ Istio core installed                                                            
✔ Istiod installed                                                                
✔ Ingress gateways installed                                                      
✔ Installation complete
```

Verify the service type of `istio-ingressgateway` service:

```
kubectl get svc istio-ingressgateway -n istio-system
```


## Step 2: Configure istio-ingressgateway Service to Use Application Load Balancer

1. Note down the `nodePort` value of istio-ingressgateway service:

    ```
    kubectl get svc istio-ingressgateway -n istio-system -o yaml
    ```

    Note down the value of `nodePort` from `.spec.ports` that corresponds to `status-port`.

    ```yaml
    ...
    spec:
      ...
      ports:
      - name: status-port
        nodePort: 30594 # Note down this value
        port: 15021
        protocol: TCP
        targetPort: 15021
    ...
    ```

2. Note down the health check path of istio-ingressgateway service:

    ```
    kubectl get deploy/istio-ingressgateway -n istio-system -o yaml
    ```

    Note down the health check path for the `readinessProbe`.

    ```yaml
    ...
    readinessProbe:
        failureThreshold: 30
        httpGet:
        path: /healthz/ready # Note down this value
        port: 15021
        scheme: HTTP
    ...
    ```

3. Edit the istio-ingressgateway service to add alb annotations:

    Edit the `istio-ingressgateway` service by adding annotations that aws application load balancer controller can use to configure health check for this target.

    !!! note
        The default kubectl editor is vim. You can change it to nano as follows:

        ```
        # Configure kubectl to use nano editor
        export KUBE_EDITOR=nano
        ```

    Use `kubectl edit` command to edit the `istio-ingressgateway` service:

    ```
    kubectl edit svc istio-ingressgateway -n istio-system
    ```

    Edit the service by adding the following annotations in `.metadata.annotations`:

    ```
    alb.ingress.kubernetes.io/healthcheck-port: "30594"
    alb.ingress.kubernetes.io/healthcheck-path: /healthz/ready
    ```

    Make sure to change the `healthcheck-port` value to the `nodePort` value you noted earlier. The same goes for `healthcheck-path`. Make sure to change it to the health check path you recorded.

Now, describe the service to check if everything is fine:

```
kubectl describe svc/istio-ingressgateway -n istio-system
```


## Step 3: Create Ingress for the istio-ingressgateway Service

Now, let's create and deploy ingress for the `istio-ingressgateway` service which in turn will create an application load balancer that sends traffic to `istio-ingressgateway` service.

=== ":octicons-file-code-16: `istio-ingressgateway-ingress.yml`"

    ```yaml linenums="1"
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: istio-ingressgateway
      namespace: istio-system
      annotations:
        # Load Balancer Annotations
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/load-balancer-name: istio-load-balancer
        # Listerner Ports Annotation
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
        # SSL Redicrect Annotation
        alb.ingress.kubernetes.io/ssl-redirect: '443'
        # SSL Certificate Annotation
        alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:170476043077:certificate/2d88e035-cde7-472a-9cd3-6b6ce6ece961
    spec:
      ingressClassName: alb
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: istio-ingressgateway
                port:
                  number: 80
    ```

Make sure to replace the `certificate-arn` with the arn of certificate you created in ACM.

Apply the manifest to create ingress:

```
# Create ingress
kubectl apply -f istio-ingressgateway-ingress.yml
```

List ingress resources:

```
# List ingress
kubectl get ing -n istio-system
```

Verify that a target group and an application load balancer was created. Also, verify that the targets are in healthy state.

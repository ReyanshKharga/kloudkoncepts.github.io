---
description: 
---

# Using Istio for Service Mesh

Services itself can't be part of the service mesh. They have to opt-in to become part of the istio service mesh.

There are two ways for the services to opt-in to become part of the service mesh:

1. Through namespace labeling
2. Injecting the sidecar proxies to pod or deployment object


## Opting In for the Mesh Through Namespace Labeling

We can add the label `istio-injection=enabled` to a namespace so that all the pods running in the namespace will become part of the service mesh.

```
kubectl label namespace <namespace-name> istio-injection=enabled
```

You can also add the label directly in the YAML when you create the namespace using declarative approach.

This approach of labelling the namespace meshes the entire namespace. Adding this namespace label instructs Istio to automatically inject Envoy sidecar proxies when you deploy your application later.

!!! note
    Istio will inject sidecar proxies only to the pods that come after adding a label to the namespace. Existing pods won't be added to mesh. You can delete and recreate the pods if you want them to become part of the service mesh.

Istio uses an extended version of the Envoy proxy which is a high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh. Envoy proxies are the only Istio components that interact with data plane traffic.

Let's test it out!

### Step 1: Create a Namespace

First, let's create a namespace as follows:

```
# Create a namespace
kubectl create ns test

# View the namespace in yaml format
kubectl get ns test -o yaml
```

### Step 2: Create Pod in the Namespace

Next, create a pod in the namespace we created above:

```
# Create a pod
kubectl run nginx --image=nginx -n test

# List pods
kubectl get pods -n test
```

You'll notice that there is only one container running in the pod.


### Step 3: Add Label to the Namespace

Now, let's add `istio-injection=enabled` label to the namespace:

```
# Add label to namespace
kubectl label namespace test istio-injection=enabled

# View the updated namespace in yaml format
kubectl get ns test -o yaml
```

See if sidecar was injected to existing pod:

```
# List pods
kubectl get pods -n test
```

Since the label was applied after the pod was created, istio won't inject the sidecar in the existing pod.

### Step 4: Delete and Recreate Pod

Delete the pod and recreate it to see if istio injects the sidecar proxies:

```
# Delete pod
kubectl delete po nginx -n test

# Recreate pod
kubectl run nginx --image=nginx -n test
```

Verify if sidecar was injected to the pod:

```
# List pods
kubectl get pods -n test
```

This time you'll see two containers running in the nginx pod because istio injected a sidecar container to our pod.

Also, view the pod definition using the following command:

```
# View the yaml definition of the pod
kubectl get pod nginx -n test -o yaml
```

On inspection, you will find that a container called istio-proxy is running in the pod. This is the sidecar container that was injected by istio.


### Step 5: Clean Up

Delete the resources we created:

```
kubectl delete ns test
```

This will delete the namespace `test` and all resources in it.




## Opting In for the Mesh by Injecting the Sidecar Proxies

You can update an existing pod or deployment by injecting the sidecar proxies using the following command:

```
kubectl get <deployment-or-pod-name> -n <namespace-name> -o yaml | istioctl kube-inject -f - | kubectl apply -f -
```

This approach enables to you inject sidecar proxies only to certain objects and not the entire namespace. However, it's less preferable due to its tedious nature—you'll need to manually inject the sidecar into each deployment object, making it less scalable and more error-prone.








---
description: Learn to seamlessly integrate ExternalDNS with Istio Gateway and Virtual Service in this step-by-step tutorial. Automate Route 53 record addition based on Istio-defined hosts effortlessly.
---

# Configure External DNS to Use Istio Gateway and Virtual Service

In this tutorial we are going to update the existing external-dns setup so that it seamlessly integrates with the istio gateway and virtual service. 

With this update ExternalDNS will automatically add records in Route 53 based on hosts defined in Istio Gateway or Virtual Service.


## Step 1: Update the ExternalDNS Manifest

Add the following arguments in the external-dns Deployment:

```
--source=istio-gateway
--source=istio-virtualservice
```

Also, in the `ClusterRole` append a new rule for istio as follows:

```yaml
- apiGroups: ["networking.istio.io"]
  resources: ["gateways", "virtualservices"]
  verbs: ["get","watch","list"]
```

The final YAML manifest file should looke like this:

=== ":octicons-file-code-16: `external-dns.yml`"

    ```yaml linenums="1"
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: external-dns
      labels:
        app.kubernetes.io/name: external-dns
    rules:
    - apiGroups: [""]
      resources: ["services", "endpoints", "pods", "nodes"]
      verbs: ["get","watch","list"]
    - apiGroups: ["extensions", "networking.k8s.io"]
      resources: ["ingresses"]
      verbs: ["get","watch","list"]
    - apiGroups: ["networking.istio.io"]
      resources: ["gateways", "virtualservices"]
      verbs: ["get","watch","list"]
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: external-dns-viewer
      labels:
        app.kubernetes.io/name: external-dns
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: external-dns
    subjects:
    - kind: ServiceAccount
      name: external-dns
      namespace: external-dns
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: external-dns
      namespace: external-dns
      labels:
        app.kubernetes.io/name: external-dns
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/name: external-dns
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app.kubernetes.io/name: external-dns
        spec:
          serviceAccountName: external-dns
          securityContext:
            fsGroup: 65534
          containers:
          - name: external-dns
            image: bitnami/external-dns:0.13.1
            # must specify env AWS_REGION in AWS china regions
            # env:
            # - name: AWS_REGION
            #   value: cn-north-1
            args:
            - --source=service
            - --source=ingress
            - --source=istio-gateway        # choose one
            - --source=istio-virtualservice # or both
            # - --domain-filter=external-dns-test.my-org.com # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
            - --provider=aws
            # - --policy=upsert-only # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
            # - --aws-zone-type=public # only look at public hosted zones (valid values are public, private or no value for both)
            - --registry=txt
            - --txt-owner-id=my-identifier
    ```


## Step 2: Apply the Manifest

Apply the manifest to update the ExternalDNS:

```
kubectl apply -f updated-external-dns.yaml
```


!!! quote "References:"
    !!! quote ""
        * [Configuring ExternalDNS to Use Istio Gateway and/or Virtual Service]{:target="_blank"}


<!-- Hyperlinks -->
[Configuring ExternalDNS to Use Istio Gateway and/or Virtual Service]: https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/istio.md
---
description: Simplified installation of ExternalDNS. Learn how to install ExternalDNS effortlessly with our step-by-step guide. Make your Kubernetes resources discoverable via public DNS servers using this helpful tool.
---

# Install ExternalDNS

Let's see how you can install `ExternalDNS` in your EKS cluster.


## Step 1: Get the Required IAM Policy

First, get the [IAM Policy]{:target="_blank"} from official git repository. It should look something like this:

=== ":octicons-file-code-16: `external-dns-iam-policy.json`"

    ```json linenums="1"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "route53:ChangeResourceRecordSets"
          ],
          "Resource": [
            "arn:aws:route53:::hostedzone/*"
          ]
        },
        {
          "Effect": "Allow",
          "Action": [
            "route53:ListHostedZones",
            "route53:ListResourceRecordSets"
          ],
          "Resource": [
            "*"
          ]
        }
      ]
    }
    ```

The above IAM policy allows `ExternalDNS` to update Route 53 Resource Record Sets and Hosted Zones. If you prefer, you may fine-tune the policy to permit updates only to explicit Hosted Zone IDs.


## Step 2: Create IAM Policy

We need to create a policy in IAM first. We will name the policy `ExternalDNSIAMPolicy`. But you can name it anything that you prefer.

```
aws iam create-policy \
    --policy-name ExternalDNSIAMPolicy \
    --policy-document file://external-dns-iam-policy.json
```

Note down the `ARN` of the policy. We'll need it in the next section.


## Step 3: Create IAM Role and Service Account

We'll use IAM Roles for Service Accounts (IRSA) to grant `ExternalDNS` permission to AWS resources. So, let's create IRSA as follows:

```
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --name external-dns \
  --namespace external-dns \
  --attach-policy-arn <policy-arn> \
  --approve
```

Please note that we have specified the namespace as `external-dns`, and as a result, the service account will be created within this namespace.

Verify the service account:

```
# List service accounts
kubectl get sa -n external-dns

# View the service account definition in yaml format
kubectl get sa external-dns -n external-dns -o yaml

# Describe the service account
kubectl describe sa external-dns -n external-dns
```

Also, go to AWS console and verify the IAM role that was created. You can get the role name from the annotation in the service account that was created.


## Step 4: Install ExternalDNS

With the service account ready, we can now move forward with the installation of `ExternalDNS`.

1. Download the YAML manifest for ExternalDNS:

    ```
    # Download external-dns manifest
    wget https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/examples/external-dns.yaml
    ```

2. Update the YAML manifest:

    Now, before we proceed with the installation of this manifest, we need to make some modifications to it.

    We'll deploy all the resources in `external-dns` namespace. So, we need to make the following modifications to ensure that resources are created in the `external-dns` namespace:

    - In `ClusterRoleBinding` object replace `namespace: default` with `namespace: external-dns` since we have created the service account in `external-dns` namespace.
    - In `Deployment` object add `namespace: external-dns` so that the resources are deployed in `external-dns` namespace
    - `ClusterRole` and `ClusterRoleBinding` are not namespaced objects so we don't have to specify the namespace

    We'll also omit the `--domain-filter`, `--policy`, and `--aws-zone-type` because we want ExternalDNS to manage all the public and private hosted zones and enable full synchronization.

    The modified manifest should look something like this:

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
                # - --domain-filter=external-dns-test.my-org.com # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
                - --provider=aws
                # - --policy=upsert-only # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
                # - --aws-zone-type=public # only look at public hosted zones (valid values are public, private or no value for both)
                - --registry=txt
                - --txt-owner-id=my-identifier
        ```


3. Apply the manifest to install ExternalDNS:

    ```
    # Install external-dns
    kubectl apply -f external-dns.yaml
    ```

4. Verify ExternalDNS pods and view logs:

    ```
    # List external-dns pods
    kubectl get pods -n external-dns

    # View external-dns logs
    kubectl logs -f <pod-name> -n external-dns
    ```


!!! quote "References:"
    !!! quote ""
        * [Required IAM Policy for External-DNS]{:target="_blank"}
        * [ExternalDNS Manifest]{:target="_blank"}


<!-- Hyperlinks -->
[IAM Policy]: https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#iam-policy
[Required IAM Policy for External-DNS]: https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#iam-policy
[ExternalDNS Manifest]: https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/examples/external-dns.yaml
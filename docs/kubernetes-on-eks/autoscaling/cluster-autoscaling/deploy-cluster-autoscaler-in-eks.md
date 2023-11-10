---
description: Deploy Kubernetes Cluster Autoscaler in Amazon EKS seamlessly. Learn the step-by-step process to dynamically scale resources based on demand.
---

# Deploy Kubernetes Cluster AutoScaler in EKS

The `Cluster Autoscaler` uses AWS scaling groups. It automatically adjusts the number of nodes in your cluster when pods fail or are rescheduled onto other nodes.

The Cluster Autoscaler is typically installed as a Deployment in your cluster.


## Prerequisites

The Cluster Autoscaler requires the following tags on your Auto Scaling groups so that they can be auto-discovered.

| Key | Value |
|----------------------------------------------|----------|
| <a>`k8s.io/cluster-autoscaler/<eks-cluster-name>`</a> | `owned` |
| <a>`k8s.io/cluster-autoscaler/enabled`</a> | `true` |

!!! note
    If you used `eksctl` to create your node groups, these tags are automatically applied.

    If you didn't use `eksctl`, you must manually tag your Auto Scaling groups with the tags mentioned above. Make sure to replace `<eks-cluster-name>` in `k8s.io/cluster-autoscaler/<eks-cluster-name>` with the actual name of your eks cluster.


## Step 1: Download the Required IAM Policy

First, get the [IAM Policy]{:target="_blank"} from official git repository. It should look something like this:

=== ":octicons-file-code-16: `cluster-autoscaler-policy.json`"

    ```json linenums="1"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:DescribeAutoScalingGroups",
            "autoscaling:DescribeAutoScalingInstances",
            "autoscaling:DescribeLaunchConfigurations",
            "autoscaling:DescribeScalingActivities",
            "autoscaling:DescribeTags",
            "ec2:DescribeInstanceTypes",
            "ec2:DescribeLaunchTemplateVersions"
          ],
          "Resource": ["*"]
        },
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:SetDesiredCapacity",
            "autoscaling:TerminateInstanceInAutoScalingGroup",
            "ec2:DescribeImages",
            "ec2:GetInstanceTypesFromInstanceRequirements",
            "eks:DescribeNodegroup"
          ],
          "Resource": ["*"]
        }
      ]
    }
    ```

The above IAM policy permits the `Cluster Autoscaler` to describe and manage Auto Scaling Groups, instances, launch configurations, scaling activities, tags, and instance types. It also grants access to describe images, get instance types from instance requirements in EC2, and describe node groups in Amazon EKS.


## Step 2: Create IAM Policy

Create the policy with the following command. You can change the value for `policy-name` to a desired value.

```
aws iam create-policy \
    --policy-name AmazonEKSClusterAutoscalerPolicy \
    --policy-document file://cluster-autoscaler-policy.json
```

Note down the `ARN` of the policy that was created. We'll need it in the next step.


## Step 3: Create IAM Role and Service Account

We'll use IAM Roles for Service Accounts (IRSA) to grant `Cluster Autoscaler` permission to AWS resources. So, let's create IRSA as follows:

```
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=cluster-autoscaler \
  --attach-policy-arn=<policy-arn> \
  --override-existing-serviceaccounts \
  --approve
```

This above command when executed will create an IAM Role and a Service Account in the EKS cluster. It will also annotate the service account with the role that it creates.

Verify the service account:

```
# List service accounts in kube-system namespace
kubectl get sa -n kube-system

# List service accounts in kube-system namespace with a filter
kubectl get sa -n kube-system | grep cluster-autoscaler

# View the service account definition in yaml format
kubectl get sa cluster-autoscaler -n kube-system -o yaml
```

Also, go to AWS console and verify the IAM role that was created. You can get the role name from the annotation in the service account that was created.


## Step 4: Deploy the Cluster Autoscaler

With the service account ready, we can now move forward and deploy `Cluster Autoscaler`.

First, download the YAML manifest for Cluster Autoscaler:

```
# Download external-dns manifest
curl -o cluster-autoscaler-autodiscover.yaml https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

Now, before we proceed with the installation of this manifest, we need to make some modifications to the YAML manifest we downloaded:

1. Replace `<YOUR CLUSTER NAME>` in container command of the `cluster-autoscaler` Deployment object with the name of your EKS cluster.

2. Modify the `cluster-autoscaler` Deployment object by adding the <a>`cluster-autoscaler.kubernetes.io/safe-to-evict: 'false'`</a> annotation in `.spec.template.metadata.annotations` section.

3. Modify the `cluster-autoscaler` container of the `cluster-autoscaler` Deployment by adding the following options:

    - `--balance-similar-node-groups`
    - `--skip-nodes-with-system-pods=false`

    The final container command of the `cluster-autoscaler` Deployment should look something like this:

    ```yaml
    command:
      - ./cluster-autoscaler
      - --v=4
      - --stderrthreshold=info
      - --cloud-provider=aws
      - --skip-nodes-with-local-storage=false
      - --expander=least-waste
      - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
      - --balance-similar-node-groups
      - --skip-nodes-with-system-pods=false
    ```

4. Get the Deployment Image tag:

    Open the Cluster Autoscaler [releases]{:target="_blank"} page from GitHub in a web browser and find the latest Cluster Autoscaler version that matches the kubernetes major and minor version of your cluster. 
    
    For example, if the kubernetes version of your cluster is `1.28`, find the latest Cluster Autoscaler release that begins with `1.28`. Record the semantic version number (`1.28.n`) for that release to use in the next step.
    
    !!! tip
        Use the search bar to search a specific version. For example you can search `Cluster Autoscaler 1.28`. Note down the semantic version number. For example `1.28.0`.

5. Modify the Deployment Image tag:

    Modify the YAML manifest file to set the `cluster-autoscaler` Deployment image tag to the version that you recorded in the previous step. In my case I have changed it to the following:

    ```
    registry.k8s.io/autoscaling/cluster-autoscaler:v1.28.0
    ```

6. Modify the YAML manifest file to omit the `cluster-autoscaler` ServiceAccount kubernetes object since we already created this using `eksctl`.

After all the modifications, the updated YAML manifest file should look something like this:

??? tip "Expand to see the updated manifest"

    === ":octicons-file-code-16: `cluster-autoscaler-autodiscover.yaml`"

        ```yaml linenums="1"
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: cluster-autoscaler
          labels:
            k8s-addon: cluster-autoscaler.addons.k8s.io
            k8s-app: cluster-autoscaler
        rules:
          - apiGroups: [""]
            resources: ["events", "endpoints"]
            verbs: ["create", "patch"]
          - apiGroups: [""]
            resources: ["pods/eviction"]
            verbs: ["create"]
          - apiGroups: [""]
            resources: ["pods/status"]
            verbs: ["update"]
          - apiGroups: [""]
            resources: ["endpoints"]
            resourceNames: ["cluster-autoscaler"]
            verbs: ["get", "update"]
          - apiGroups: [""]
            resources: ["nodes"]
            verbs: ["watch", "list", "get", "update"]
          - apiGroups: [""]
            resources:
              - "namespaces"
              - "pods"
              - "services"
              - "replicationcontrollers"
              - "persistentvolumeclaims"
              - "persistentvolumes"
            verbs: ["watch", "list", "get"]
          - apiGroups: ["extensions"]
            resources: ["replicasets", "daemonsets"]
            verbs: ["watch", "list", "get"]
          - apiGroups: ["policy"]
            resources: ["poddisruptionbudgets"]
            verbs: ["watch", "list"]
          - apiGroups: ["apps"]
            resources: ["statefulsets", "replicasets", "daemonsets"]
            verbs: ["watch", "list", "get"]
          - apiGroups: ["storage.k8s.io"]
            resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
            verbs: ["watch", "list", "get"]
          - apiGroups: ["batch", "extensions"]
            resources: ["jobs"]
            verbs: ["get", "list", "watch", "patch"]
          - apiGroups: ["coordination.k8s.io"]
            resources: ["leases"]
            verbs: ["create"]
          - apiGroups: ["coordination.k8s.io"]
            resourceNames: ["cluster-autoscaler"]
            resources: ["leases"]
            verbs: ["get", "update"]
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
          name: cluster-autoscaler
          namespace: kube-system
          labels:
            k8s-addon: cluster-autoscaler.addons.k8s.io
            k8s-app: cluster-autoscaler
        rules:
          - apiGroups: [""]
            resources: ["configmaps"]
            verbs: ["create","list","watch"]
          - apiGroups: [""]
            resources: ["configmaps"]
            resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
            verbs: ["delete", "get", "update", "watch"]

        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: cluster-autoscaler
          labels:
            k8s-addon: cluster-autoscaler.addons.k8s.io
            k8s-app: cluster-autoscaler
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: cluster-autoscaler
        subjects:
          - kind: ServiceAccount
            name: cluster-autoscaler
            namespace: kube-system

        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: RoleBinding
        metadata:
          name: cluster-autoscaler
          namespace: kube-system
          labels:
            k8s-addon: cluster-autoscaler.addons.k8s.io
            k8s-app: cluster-autoscaler
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: Role
          name: cluster-autoscaler
        subjects:
          - kind: ServiceAccount
            name: cluster-autoscaler
            namespace: kube-system

        ---
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: cluster-autoscaler
          namespace: kube-system
          labels:
            app: cluster-autoscaler
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: cluster-autoscaler
          template:
            metadata:
              labels:
                app: cluster-autoscaler
              annotations:
                cluster-autoscaler.kubernetes.io/safe-to-evict: 'false'
                prometheus.io/scrape: 'true'
                prometheus.io/port: '8085'
            spec:
              priorityClassName: system-cluster-critical
              securityContext:
                runAsNonRoot: true
                runAsUser: 65534
                fsGroup: 65534
              serviceAccountName: cluster-autoscaler
              containers:
                - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.28.0
                  name: cluster-autoscaler
                  resources:
                    limits:
                      cpu: 100m
                      memory: 600Mi
                    requests:
                      cpu: 100m
                      memory: 600Mi
                  command:
                    - ./cluster-autoscaler
                    - --v=4
                    - --stderrthreshold=info
                    - --cloud-provider=aws
                    - --skip-nodes-with-local-storage=false
                    - --expander=least-waste
                    - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
                    - --balance-similar-node-groups
                    - --skip-nodes-with-system-pods=false
                  volumeMounts:
                    - name: ssl-certs
                      mountPath: /etc/ssl/certs/ca-certificates.crt #/etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
                      readOnly: true
                  imagePullPolicy: "Always"
              volumes:
                - name: ssl-certs
                  hostPath:
                    path: "/etc/ssl/certs/ca-bundle.crt"
        ```

Now, apply the modified manifest file in your EKS cluster to deploy the Cluster Autoscaler:

```
kubectl apply -f cluster-autoscaler-autodiscover.yaml
```


## Step 5: View Cluster Autoscaler logs

After you have deployed the Cluster Autoscaler, you can view the logs and verify that it's monitoring your cluster load.

View your Cluster Autoscaler logs using the following command:

```
kubectl logs -f deployment.apps/cluster-autoscaler -n kube-system
```

The output should look something like this:

```yaml
I1205 06:53:27.855003       1 static_autoscaler.go:230] Starting main loop
I1205 06:53:27.855390       1 filter_out_schedulable.go:65] Filtering out schedulables
I1205 06:53:27.855400       1 filter_out_schedulable.go:132] Filtered out 0 pods using hints
I1205 06:53:27.855405       1 filter_out_schedulable.go:170] 0 pods were kept as unschedulable based on caching
I1205 06:53:27.855409       1 filter_out_schedulable.go:171] 0 pods marked as unschedulable can be scheduled.
I1205 06:53:27.855414       1 filter_out_schedulable.go:82] No schedulable pods
I1205 06:53:27.855423       1 static_autoscaler.go:419] No unschedulable pods
I1205 06:53:27.855433       1 static_autoscaler.go:466] Calculating unneeded nodes
I1205 06:53:27.855444       1 pre_filtering_processor.go:66] Skipping ip-192-168-100-52.ap-south-1.compute.internal - node group min size reached
I1205 06:53:27.855449       1 pre_filtering_processor.go:66] Skipping ip-192-168-71-21.ap-south-1.compute.internal - node group min size reached
I1205 06:53:27.855467       1 static_autoscaler.go:520] Scale down status: unneededOnly=false lastScaleUpTime=2022-12-05 05:49:57.069585724 +0000 UTC m=-3579.325882549 lastScaleDownDeleteTime=2022-12-05 05:49:57.069585724 +0000 UTC m=-3579.325882549 lastScaleDownFailTime=2022-12-05 05:49:57.069585724 +0000 UTC m=-3579.325882549 scaleDownForbidden=false isDeleteInProgress=false scaleDownInCooldown=false
I1205 06:53:27.855487       1 static_autoscaler.go:533] Starting scale down
I1205 06:53:27.855513       1 scale_down.go:918] No candidates for scale down
```


!!! quote "References:"
    !!! quote ""
        * [Cluster Autoscaler YAML For AWS]{:target="_blank"}
        * [Cluster Autoscaler IAM Policy]{:target="_blank"}



<!-- Hyperlinks -->
[IAM Policy]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#iam-policy
[releases]: https://github.com/kubernetes/autoscaler/releases
[Cluster Autoscaler YAML For AWS]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
[Cluster Autoscaler IAM Policy]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#iam-policy
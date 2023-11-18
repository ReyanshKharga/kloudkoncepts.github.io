---
description: Learn how to efficiently create a private Amazon EKS NodeGroup with our step-by-step guide. Secure your Kubernetes cluster today and streamline your infrastructure management.
---

# Create Private EKS NodeGroup

So far, our focus has been on using public NodeGroups. However, in a production environment, it is crucial to ensure the security of your nodes by making them private and restricting public access.

From now on, our approach will involve using private nodegroups as the preferred method.

First, we will create a private nodegroup, and subsequently, we will delete the existing public nodegroup.


## Step 1: Verify Existing NodeGroups and Nodes

Make sure the AWS CLI is configured and the profile is exported if you are using a named profile:

```
export AWS_PROFILE=<aws-profile-name>
```

Verify existing nodegroups and nodes:

```
# List nodegroups
eksctl get nodegroups --cluster <cluster-name>

# List worker nodes
kubectl get nodes
```


## Step 2: Create a Private EKS NodeGroup

We will use `eksctl` to create a private NodeGroup.

We will use a configuration file since it requires numerous parameters, although you can also do it via the command line.

You can reuse the `cluster.yml` file we used earlier to create cluster and public EKS nodegroup. Simply apply the following modifications:

1. Make a copy of `cluster.yml` and name it anything you like. Let's name it `private-nodegroup.yml`.
2. Remove the `version` field from the metadata object. We only need cluster `name` and `region`.
3. Remove the top-level `iam` object. This is needed only when we create the cluster.
4. In the `managedNodeGroups` change the `name` field. Let's name it `private-nodegroup`.
5. In the `managedNodeGroups` change the `privateNetworking` field to `true` since we want our worker nodes to be present in private subnets.

The modified file should look similar to the below:

=== ":octicons-file-code-16: `private-nodegroup.yml`"

    ```yaml linenums="1"
    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig

    metadata:
      name: my-cluster
      region: ap-south-1
      # version: "1.28"

    availabilityZones:
      - ap-south-1a
      - ap-south-1b
      - ap-south-1c

    # iam:
    #   withOIDC: true

    managedNodeGroups:
      - name: private-nodegroup
        privateNetworking: true
        instanceType: t3.medium
        minSize: 2
        maxSize: 2
        volumeSize: 20
        ssh:
          allow: true
          publicKeyName: my-eks-key
        iam:
          withAddonPolicies:
            imageBuilder: true
            autoScaler: true
            externalDNS: true
            certManager: true
            appMesh: true
            appMeshPreview: true
            ebs: true
            fsx: true
            efs: true
            awsLoadBalancerController: true
            xRay: true
            cloudWatch: true
    ```

Apply the config to create private nodegroup in our eks cluster:

```
# Create private nodegroup
eksctl create nodegroup --config-file=private-nodegroup.yml
```


## Step 3: Verify the Private NodeGroup and Nodes

```
# List nodegroups
eksctl get nodegroups --cluster <cluster-name>

# List worker nodes
kubectl get nodes
```


## Step 4: Delete Public Nodes and NodeGroup

Once the nodes from the private nodegroup are in `Ready` state we can go ahead and delete our public nodegroup.

To safely delete the public nodegroup in your Amazon EKS cluster, follow these steps:

1. Confirm that all necessary applications and services are running smoothly:

    ```
    # List pods in all namespaces
    kubectl get pods -A -o wide

    # List services in all namespaces
    kubectl get svc -A
    ```

2. Cordon the public nodes to prevent new pods from being scheduled on them. You can use the following command to cordon each node in the public nodegroup:

    ```
    kubectl cordon <node_name>
    ```

3. Drain the public nodes to gracefully evict any running workloads. You can use the following command to drain each node:

    ```
    kubectl drain <node_name> --ignore-daemonsets --delete-emptydir-data
    ```

4. Verify that all pods (except Daemonset pods) have been successfully moved to the private nodes by running the following command:

    ```
    # List nodes
    kubectl get nodes

    # List pods
    kubectl get pods -A -o wide
    ```

5. Once all pods have been evacuated, you can delete the public nodegroup safely:

    ```
    # Delete public nodegroup
    eksctl delete nodegroup <public-nodegroup-name> --cluster <cluster-name>
    ```

6. Verify nodes and nodegroups:

    ```
    # List nodegroups
    eksctl get nodegroups --cluster <cluster-name>

    # List worker nodes
    kubectl get nodes
    ```
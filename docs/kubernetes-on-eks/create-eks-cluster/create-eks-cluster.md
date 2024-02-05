---
description: Learn how to create an Amazon EKS cluster effortlessly using eksctl. Step-by-step guide for quick, hassle-free cluster setup and management.
---

# Create EKS Cluster eksctl

## Prerequisites

- Make sure you have `AWS CLI`, `kubectl`, and `eksctl` installed.
- Make sure you have configured the `AWS CLI`.
- Make sure the IAM user has Administrator access.

!!! warning
    It's important to note that while we've simplified the process by granting Administrator access for convenience, it's generally recommended to provide more granular and specific permissions to IAM users based on their actual needs. This helps reduce potential security risks and ensures a more controlled and secure environment.


## Methods of Using eksctl

There are two methods of using eksctl:

1. **Imperative method:** 

    This method involves running `eksctl` commands directly in the terminal to create, modify, or delete EKS resources. For example, you can use the following command to create a new EKS cluster:

    ```
    eksctl create cluster --name my-cluster --region ap-south-1
    ```

    The above command creates a new EKS cluster with the name `my-cluster` in the `ap-south-1` region.

2. **Declarative method:**

    This method involves defining a configuration file in YAML format that specifies the desired state of the EKS cluster and using `eksctl` to apply that configuration. 
    
    For example, you can define a configuration file called `cluster.yaml` that creates an EKS cluster with two worker nodes as follows:

    === ":octicons-file-code-16: `cluster.yml`"

    ```yaml linenums="1"
    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig

    metadata:
      name: my-cluster
      region: ap-south-1

    nodeGroups:
      - name: my-nodegroup
        instanceType: t2.small
        desiredCapacity: 2
    ```

    You can then apply this configuration by running the following command:

    ```
    eksctl create cluster -f cluster.yaml
    ```

    The above command creates a new EKS cluster with the name `my-cluster` in the `ap-south-1` region, and a worker node group with two `t2.small` instances.

    We will be using the declarative method in this course as it is much easier to specify complex parameters using configuration files, compared to the tedious task of specifying the same parameters directly through the command line.


## Amazon EKS Pricing

Heads up! If you're following along with this course on Amazon EKS, be aware that you will incur costs for the EKS infrastructure.

Here are some tips for understanding EKS pricing:

- EKS cluster management is charged at $0.10 per hour per cluster, regardless of the number of nodes in the cluster.
- EC2 instances are used as worker nodes in an EKS cluster, and you will be charged for the hours these instances are running.
- EKS also uses other AWS services like load balancers, storage, and networking, which will incur additional costs.
- Be sure to monitor your AWS billing dashboard to avoid unexpected costs and set up billing alerts to stay informed about your usage.


## Step 1: Create Key Pair (Optional)

This step is required only if you want to enable `SSH` for worker nodes in the EKS cluster.

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)
2. In the navigation pane, under Network & Security, choose Key Pairs.
3. Choose Create key pair
4. Enter the desired name for the key pair (For example, `my-eks-key`)
5. For Key pair type, choose `RSA`
6. Add tags if you want
7. Click on Create key pair

The private key will be downloaded. Keep it safely.


## Step 2: Prepare Configuration File

Prepare configuration file in `YAML` format to specify the desired state of the EKS cluster.

=== ":octicons-file-code-16: `cluster.yml`"

    ```yaml linenums="1"
    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig

    metadata:
      name: my-cluster
      region: ap-south-1
      version: "1.28"

    availabilityZones:
      - ap-south-1a
      - ap-south-1b
      - ap-south-1c

    iam:
      withOIDC: true

    managedNodeGroups:
      - name: public-nodegroup
        privateNetworking: false
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

Make sure to make the following changes in the above `cluster.yml` configuration file:

1. Replace the value of `publicKeyName` with the name of the key pair you created in `Step 1`.
2. Remove the `ssh` parameter if you don't want to enable `SSH` for worker nodes.
3. Replace the `region` and `availabilityZones` fields if you are working in a different region.

The provided configuration, when applied, will create an Amazon Elastic Kubernetes Service (EKS) cluster in the `ap-south-1` region with the name `my-cluster`. This cluster will span multiple availability zones: `ap-south-1a`, `ap-south-1b`, and `ap-south-1c`. It will also enable IAM (Identity and Access Management) with OIDC (OpenID Connect) for authentication.

Additionally, the configuration specifies a managed node group named `public-nodegroup` with the following settings: instance type `t3.medium`, a fixed group size of 2 instances, each with a 20GB volume. `SSH` access is allowed, and it uses the specified public key `my-eks-key` for authentication. 

The IAM role for this node group is configured with various addon policies enabled, including features like image builder, auto-scaling, external DNS, certificate manager, App Mesh, App Mesh Preview, EBS (Elastic Block Store), FSx (Amazon FSx for Lustre), EFS (Elastic File System), AWS Load Balancer Controller, X-Ray, and CloudWatch.

!!! note
    - The above cofiguration will create a dedicated VPC for the cluster but you can also configure it to [use an existing VPC]{:target="_blank"}.

    - You may choose to skip creating node groups and IAM OIDC provider at this stage and create it later. But we are creating them at this stage to have everything set up so that we can start using the Kubernetes cluster without additional effort.



## Step 3: Set the AWS_PROFILE Environment Variable

Set the `AWS_PROFILE` environment variable if you are using a named aws profile as follows:

```
# Command template
export AWS_PROFILE=<my-aws-profile>

# Actual command
export AWS_PROFILE=eks-profile
```

Verify that the AWS CLI is configured properly.

```
# Verify profile
aws configure list
```


## Step 4: Create Cluster

Use `eksctl` to apply the `cluster.yml` configuration and create cluster.

```
# Command template to create EKS cluster
eksctl create cluster -f <file-name>

# Actual command
eksctl create cluster -f cluster.yml
```

This will create the EKS cluster and also update the kube config for the cluster in `~/.kube/config`.

Kubeconfig is a configuration file used by Kubernetes to manage access to Kubernetes clusters. It is used by the Kubernetes command-line tool, `kubectl`, to connect to a Kubernetes cluster and perform administrative tasks.

The `kubeconfig` file contains information about one or more Kubernetes clusters, the authentication credentials to access them, and the context of a user and namespace.

In Kubernetes, a context is a way to set and switch between different Kubernetes clusters, users, and namespaces. Each context contains the configuration information needed to communicate with a specific Kubernetes cluster.

View the kube config file:

```
cat ~/.kube/config
```

The `kubeconfig` file will be valid for 24 hours by default.

You can update the kube config using the following command if needed:

```
# Command template
aws eks update-kubeconfig --name <eks-cluster-name> --region <region-name>

# Actual command
aws eks update-kubeconfig --name my-cluster --region ap-south-1
```

This will update the kube config and will be valid for 24 hours.

View the name of the current context that `kubectl` is using:

```
# Display the current context
kubectl config current-context
```


## Step 5: Verify EKS Cluster in the AWS Console

Login to AWS console and verify that the cluster was created.

!!! warning
    Make sure you are logged in with the same IAM user that you used to create the cluster or else you won't be able to see some of the details.

You can also use `eksctl` or `AWS CLI` to list clusters as follows:

```
# Use eksctl to list clusters
eksctl get cluster

{OR}

# Use AWS CLI to list clusters
aws eks list-clusters
```


## Step 6: Verify CloudFormation Stack

When you use `eksctl` to create an Amazon EKS cluster, it sets up a CloudFormation stack set, which in turn is used to create required AWS resources such as the EKS cluster, VPC, IAM roles, EC2 instances, etc.

Go to AWS Console and verify the stack set created by `eksctl`.


!!! quote "References:"
    !!! quote ""
        * [Getting started with eksctl]{:target="_blank"}
        * [Creating and managing clusters using eksctl]{:target="_blank"}
        * [Amazon EKS Pricing]{:target="_blank"}


<!-- Hyperlinks -->
[Creating and managing clusters using eksctl]: https://eksctl.io/usage/creating-and-managing-clusters/
[Getting started with eksctl]: https://eksctl.io/getting-started/
[Use an existing VPC]: https://eksctl.io/usage/vpc-configuration/#use-existing-vpc-other-custom-configuration
[Amazon EKS Pricing]: https://aws.amazon.com/eks/pricing/
---
description: Install the EBS CSI Driver effortlessly in your kubernetes cluster with our step-by-step guide.
---

# Install EBS CSI Driver

Let's install the Amazon `EBS CSI Driver` in our EKS cluster which will enable us to use Amazon EBS volumes as persistent volumes.


## Step 1: Verify or Create the IAM OIDC provider

The Amazon `EBS CSI driver` requires the use of the IAM OpenID Connect (OIDC) provider for authentication and authorization.

This means that before installing the driver, we need to ensure that the IAM OIDC provider is created for your EKS cluster.

We have the IAM OIDC provider already created when we created the cluster using `eksctl`.

Let's verify if we have the IAM OIDC provider created for the cluster.

1. Retrieve your cluster's OIDC provider ID and store it in a variable:

    ```
    # Command template
    oidc_id=$(aws eks describe-cluster --name <cluster-name> --region <region-name> --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

    # Actual command
    oidc_id=$(aws eks describe-cluster --name my-cluster --region ap-south-1 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
    ```

    If the AWS profile being used already has the region configured, there is no requirement for you to provide the `--region` option.

2. Determine whether an IAM OIDC provider with your cluster's ID is already in your account:

    ```
    aws iam list-open-id-connect-providers | grep $oidc_id
    ```

    If output is returned from the above command, then you already have a provider for your cluster and you can skip the next step.

3. Create IAM OIDC provider for your cluster:

    If no output is returned in the previous command, then you must create an IAM OIDC provider for your cluster. 

    ```
    # Command template
    eksctl utils associate-iam-oidc-provider --cluster <cluster-name> --region <region-name> --approve

    # Actual command
    eksctl utils associate-iam-oidc-provider --cluster my-cluster --region ap-south-1 --approve
    ```


## Step 2: Install the Amazon EBS CSI Driver

Let's install the latest `EBS CSI Driver` usign the following command:

```
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
```

Note that we are using `-k` here and not `-f`. The `kubectl apply -k` command is used to apply a set of resources defined in a directory. It allows you to define the configuration for a set of resources using a `kustomization.yaml` file in the directory.

`Kustomize` is a tool built into kubernetes that allows you to customize and manage configuration files for kubernetes applications. It simplifies the management of configuration files, allows you to version them, and provides a way to customize them for different environments.

!!! tip
    You can also install `aws-ebs-csi-driver` as `add-on` from the AWS console or using `eksctl` CLI.


## Step 3: Verify EBS CSI Driver Installation

Let's verify if `EBS CSI Driver` is installed in our EKS cluster:

```
# Get all kubernetes objects in kube-system namespace
kubectl get all -n kube-system
```

You will see the resources created by `EBS CSI Driver`.



## What is kubernetes-sigs GitHub organization (Bonus Information)

The [kubernetes-sigs]{:target="_blank"} GitHub organization is a collection of Kubernetes Special Interest Groups (SIGs) that focus on specific areas of the kubernetes project. 

These SIGs are responsible for developing and maintaining specific features, subsystems, and aspects of the Kubernetes ecosystem.

Some examples of popular repositories hosted by `kubernetes-sigs` include:

- `kustomize`: A tool for customizing kubernetes configuration files.
- `cert-manager`: A kubernetes add-on for managing and automating TLS certificates.
- `cluster-api`: A kubernetes subproject for declaratively managing infrastructure on cloud providers.
- `aws-load-balancer-controller`: A kubernetes controller that manages Application Load Balancers (ALBs) on AWS.


!!! quote "References:"
    !!! quote ""
        * [Install EBS CSI Driver]{:target="_blank"}

<!-- Hyperlinks -->
[kubernetes-sigs]: https://github.com/kubernetes-sigs
[Install EBS CSI Driver]: https://github.com/kubernetes-sigs/aws-ebs-csi-driver
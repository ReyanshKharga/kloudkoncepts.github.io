---
description: Learn how to install AWS Load Balancer Controller with our step-by-step guide. Simplify the process of setting up load balancing for your Kubernetes infrastructure on Amazon EKS. 
---

# Install AWS Load Balancer Controller

As discussed earlier, for the Ingress resource to work correctly, it is necessary to have an Ingress controller running in the cluster.

So, let's go ahead and install AWS Load Balancer Controller which is an ingress controller.


## Step 1: Verify or Create the IAM OIDC provider

The Amazon `AWS Load Balancer Controller` requires the use of the IAM OpenID Connect (OIDC) provider for authentication and authorization.

This means that before installing the controller, we need to ensure that the IAM OIDC provider is created for your EKS cluster.

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


## Step 2: Add the EKS repository to Helm

We'll be using Helm to install the AWS Load Balancer Controller. Therefore, please ensure that you have Helm installed on your local system.

Let's add the eks repository to helm:

```
helm repo add eks https://aws.github.io/eks-charts
```


## Step 3: Install AWS Load Balancer Controller

Now, that we have added the eks repository to helm, we can go ahead and install the AWS Load Balancer Controller using helm.

The Load Balancer Controller requires to have certain AWS IAM permissions since it will create load balancer and related resources in AWS account at a later stage when we create Ingress Kubernetes object.

The permissions can be granted to the AWS Load Balancer Controller either using a Service Account or using the IAM Role attached to worker nodes.

### Case 1: If not using IAM roles for service account

In this case, the AWS Load Balancer Controller inherits the required permissions from the role attached to worker nodes.

!!! note
    Since granting permissions via worker nodes is not recommended we won't use this method. We'll use the method discussed in Case 2.

1. Make sure the IAM role attached to worker nodes has the following IAM permissions at minimum:

    ```
    https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/main/docs/install/iam_policy.json
    ```

    You can also download the IAM policy from the official github repository as follows:

    ```
    curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
    ```

2. Install the AWS Load Balancer Controller using the following command:

    ```
    # Command template to install AWS Load Balancer Controller
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=<cluster-name> -n aws-load-balancer-controller

    # Actual command
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=my-cluster -n aws-load-balancer-controller
    ```

### Case 2: If using IAM roles for service account

In this case, the AWS Load Balancer Controller gains the required permissions from the service account.

1. Download IAM policy required for the AWS Load Balancer Controller:

    ```
    curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
    ```

2. Create an IAM policy:

    ```
    aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam-policy.json
    ```

3. Create an IAM role and ServiceAccount for the Load Balancer controller:

    Let's create an IAM role and service account. Use the policy `ARN` from the previous step. If you prefer a predetermined role name you can specify `--role-name` parameter. (We'll let `eksctl` name the role)

    ```
    # Create IAM role and service account
    eksctl create iamserviceaccount \
    --cluster=<cluster-name> \
    --namespace=aws-load-balancer-controller \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
    --approve
    ```

    This will also annotate the service account with the IAM role it creates.

4. Verify the IAM role and service account:

    ```
    # List service accounts in aws-load-balancer-controller namespace
    kubectl get sa -n aws-load-balancer-controller

    # View the service account definition in yaml format
    kubectl get sa aws-load-balancer-controller -n aws-load-balancer-controller -o yaml
    ```

    Note down the role name displayed in the `.metadata.annotations` section of the service account definition. Go to the AWS console and verify if the role exists.

5. Install the Load Balancer Controller:

    If you are using IAM Roles for service account you need to specify both of the chart values `serviceAccount.create=false` and `serviceAccount.name=aws-load-balancer-controller`.

    ```
    # Command template to install AWS Load Balancer Controller
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=<cluster-name> -n aws-load-balancer-controller --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

    # Actual Command
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=my-cluster -n aws-load-balancer-controller --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
    ```

    If the command executes successfully you will see an ouput that looks something like this:

    ```yaml
    NAME: aws-load-balancer-controller
    LAST DEPLOYED: Wed Jun 28 00:17:46 2023
    NAMESPACE: aws-load-balancer-controller
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    AWS Load Balancer controller installed!
    ```


## Step 4: Verify Resources Created by AWS Load Balancer Controller

```
# List pods
kubectl get po -n aws-load-balancer-controller

# List ingress class
kubectl get ingressclass -n aws-load-balancer-controller
```

In kubernetes, an `Ingress Class` is a way to differentiate and configure multiple Ingress controllers within a cluster. It allows you to specify different behaviors and settings for different Ingress controllers based on their class.

For example, you can have both the NGINX Ingress Controller and the AWS Load Balancer Controller installed in the same Kubernetes cluster to meet different requirements.

When creating an Ingress resource, you can specify the desired Ingress class using the `kubernetes.io/ingress.class` annotation.

Check AWS Load Balancer Controller logs:

```
kubectl logs -f deploy/aws-load-balancer-controller -n aws-load-balancer-controller
```


## Uninstall AWS Load Balancer Controller

You need to execute the following commands only when you no longer need the AWS Load Balancer Controller installed in your kubernetes cluster:

```
# Uninstall AWS Load Balancer Controller
helm uninstall aws-load-balancer-controller -n aws-load-balancer-controller

# Delete the service account
eksctl delete iamserviceaccount --name aws-load-balancer-controller --cluster <cluster-name>
```


!!! quote "References:"
    !!! quote ""
        * [AWS Load Balancer Controller Concepts]{:target="_blank"}
        * [AWS Load Balancer Controller Installation Guide]{:target="_blank"}
        * [AWS Load Balancer Controller - Git Repo]{:target="_blank"}


<!-- Hyperlinks -->
[AWS Load Balancer Controller Concepts]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/
[AWS Load Balancer Controller Installation Guide]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/deploy/installation/
[AWS Load Balancer Controller - Git Repo]: https://github.com/kubernetes-sigs/aws-load-balancer-controller/tree/main/helm/aws-load-balancer-controller
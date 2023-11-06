---
description: Learn how to create and effectively manage IAM Roles for Service Accounts in our step-by-step guide. Gain control over access permissions and security in your EKS cluster with simple, hands-on instructions.
---

# Create and Manage IAM Roles for Service Accounts

Let's see how we can create and manage IAM Roles for Service Accounts (IRSA).


## Step 1: Create a Service Account

Let's create a ordinary service account in the default namespace.

=== ":octicons-file-code-16: `my-ordinary-service-account.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: my-ordinary-service-account
    ```

Apply the manifest to create the service account:

```
kubectl apply -f my-ordinary-service-account.yml
```

Verify the service account:

```
kubectl get sa
```


## Step 2: Create Pod With Service Account

Let's create pods that uses the ordinary service account we created. We'll use deployment to create pods:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: aws-cli
      template:
        metadata:
          labels:
            app: aws-cli
        spec:
          serviceAccountName: my-ordinary-service-account
          containers:
          - name: aws-cli
            image: amazon/aws-cli
            command: ["sh", "-c",  "sleep 3600"]
    ```

Apply the manifest to create deployment:

```
kubectl apply -f my-deployment-with-ordinary-sa.yml
```

Verify Deployment and Pods:

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: Access AWS Resources From Within Pod

Let's try to list S3 buckets from within a pod in the deployment:

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# Verify if the aws-cli is installed
aws --version

# List S3 buckets
aws s3 ls
```

You'll receive the following error:

```
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```

This is because the pod doesn't have permission to access S3 buckets.


## Step 4: Create IAM Role for Service Account

We will use `eksctl` to create an IAM Role for the service account.

You have the flexibility to either provide all the parameters directly in the command line or use the `--config-file` option to supply the parameters in the `eksctl` command.

- **Create IRSA without config file**

    ```
    eksctl create iamserviceaccount \
        --name service-account-for-s3-access \
        --namespace default \
        --cluster my-cluster \
        --role-name iam-role-for-service-account-with-s3-access \
        --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess \
        --override-existing-serviceaccounts \
        --approve
    ```

- **Create IRSA using config file**

    First we need to create the config file as follows:

    === ":octicons-file-code-16: `irsa.yml`"

        ```yaml linenums="1"
        apiVersion: eksctl.io/v1alpha5
        kind: ClusterConfig

        metadata:
          name: my-cluster
          region: ap-south-1

        iam:
          withOIDC: true # Must be enabled explicitly for iam.serviceAccounts to be created
          serviceAccounts:
          - metadata:
              name: service-account-for-s3-access
              namespace: default
            roleName: iam-role-for-service-account-with-s3-access
            attachPolicyARNs:
            - arn:aws:iam::aws:policy/AmazonS3FullAccess
        ```

    Now, we can create the service account using eksctl as follows:

    ```
    eksctl create iamserviceaccount --config-file=irsa.yml --override-existing-serviceaccounts --approve
    ```

This will do the following for you:

1. Create an IAM Role in AWS
2. Create a service account in your Kubernetes cluster
3. Annotate the service account with the IAM Role ARN

Visit the AWS console and verify the IAM Role. Pay close attention to the trust policy associated with the IAM Role.

The trust policy enables the IAM OIDC provider to assume a role with web identity, but only for the specified service account.

Also, verify the service account created:

```
# List service accounts
kubectl get sa

# Describe the service account
kubectl describe sa <service-account-name>
```

You'll observe the service account we created has `eks.amazonaws.com/role-arn` annotation.


## Step 5: Update the Service Account Name in Deployment

Let's update the deployment to use the newly created service account that is associated with an IAM role granting S3 permissions.

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: aws-cli
      template:
        metadata:
          labels:
            app: aws-cli
        spec:
          serviceAccountName: service-account-for-s3-access
          containers:
          - name: aws-cli
            image: amazon/aws-cli
            command: ["sh", "-c",  "sleep 3600"]
    ```

Apply the manifest to update the deployment:

```
kubectl apply -f my-deployment.yml
```

Verify deployment and pods:

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 6: Retry Accessing AWS Resources from Within a Pod

```
# Start a shell session inside the container
kubectl exec -it <pod-name> -- bash

# Verify if the aws-cli is installed
aws --version

# List S3 buckets
aws s3 ls
```

This time, you will notice that you are able to list the S3 buckets because the service account associated with the pod is connected to an IAM Role that provides the necessary access to S3 buckets.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-ordinary-service-account.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

Also, delete IAM Role for Service Account (IRSA) that we created:

```
# Without config file
eksctl delete iamserviceaccount --name service-account-for-s3-access --cluster my-cluster

{OR}

# Using config file
eksctl delete iamserviceaccount --config-file=irsa.yml --approve
```

This will delete both the IAM Role and the service account that were created.
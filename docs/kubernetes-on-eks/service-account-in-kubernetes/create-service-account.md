---
description: Effortlessly create and manage Service Accounts in Kubernetes with our step-by-step guide. Learn how to customize identity and access control for your workloads.
---

# Create and Manage Service Accounts

Let's see how we can create and manage Service Accounts.


## Step 1: Create a Service Account

Let's create a service account in the `default` namespace:

=== ":octicons-file-code-16: `my-service-account.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: my-service-account
    ```

Apply the manifest to create the Service Account:

```
kubectl apply -f my-service-account.yml
```


## Step 2: List Service Accounts

List service accounts in the `default` namespace:

```
kubectl get sa
{OR}
kubectl get serviceaccount
{OR}
kubectl get serviceaccounts
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. The following commands produce the same output:

```
kubectl get sa 
kubectl get serviceaccount
kubectl get serviceaccounts
```


!!! note
    `serviceaccount` is abbreviated as `sa`.


## Step 3: Describe a Service Account

```
kubectl describe serviceaccount my-service-account
{OR}
kubectl describe serviceaccount/my-service-account
```

!!! note
    In kubernetes versions `1.23` and earlier, when a `service account` is created, a `secret` is also generated to store the associated `token`.

    Starting from kubernetes version `1.24`, a significant change is implemented regarding service accounts and token generation. Unlike previous versions, service accounts no longer generate tokens as secrets by default.


## Step 4: Delete a Service Account

Delete the service account:

```
kubectl delete serviceaccount my-service-account
{OR}
kubectl delete serviceaccount/my-service-account
```
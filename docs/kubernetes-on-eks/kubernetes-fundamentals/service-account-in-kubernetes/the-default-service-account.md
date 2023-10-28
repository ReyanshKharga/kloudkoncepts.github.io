---
description: Explore the default Service Account in Kubernetes and its role in workload identity management. Learn how it simplifies access control for your applications.
---

# The Default Service Account

Every namespace has a `default` service account. If you don't specify a different service account for a pod within a namespace, it will automatically use the `default` service account for API access and resource interactions.


## Step 1: List Service Accounts in Default Namespace

```
kubectl get sa
{OR}
kubectl get serviceaccount
{OR}
kubectl get serviceaccounts
```

You'll see a service account named `default`.


## Step 2: Create a Namespace and Verify the Default Service Account

Now, let's create a namespace called `dev` and list service accounts in that namespace:

```
# Create namespace dev
kubectl create ns dev

# List service accounts in dev namespace
kubectl get sa -n dev
```

You'll see a service account named `default` in the `dev` namespace.


## Clean Up

Delete the `dev` namespace we created:

```
kubectl delete ns dev
```
---
description: Effortlessly create and manage secrets in Kubernetes with our comprehensive guide. Learn how to securely handle sensitive data and credentials for your applications.
---

# Create and Manage Secrets

Let's see how we can create and manage Secrets.


## Step 1: Create a Secret

=== ":octicons-file-code-16: `my-secret.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Secret
    metadata:  
      name: my-secret
    data:
      username: cmV5YW5zaA==
      password: bXlkYnBhc3N3b3Jk
    ```

Apply the manifest to create the Secret:

```
kubectl apply -f my-secret.yml
```


## Step 2: List Secrets

```
kubectl get secrets
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get secret
kubectl get secrets
```


## Step 3: Describe a Secret

```
# Command template
kubectl describe secret <secret-name>
{OR}
kubectl describe secret/<secret-name>

# Actual command
kubectl describe secret my-secret
{OR}
kubectl describe secret/my-secret
```


## Step 4: Delete a Secret

```
# Command template
kubectl delete secret <secret-name>
{OR}
kubectl delete secret/<secret-name>

# Actual command
kubectl delete secret my-secret
{OR}
kubectl delete secret/my-secret
```


You can also use the manifest to delete the resource as follows:

```
kubectl delete -f my-secret.yml
```
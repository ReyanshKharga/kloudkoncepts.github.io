---
description: Learn how to pause and resume a deployment rollout with our step-by-step guide. Master the art of controlling and managing deployment progress in Kubernetes.
---

# Pause and Resume a Deployment Rollout

Pausing and resuming a deployment rollout is a common practice in software development when deploying new changes to production environments.

This allows you to monitor the deployment closely and address any issues that may arise before continuing with the rollout.

Let's see how you can pause and resume a deployment rollout.


## Step 1: Create a Deployment

First, let's create a Deployment as follows:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
            tier: backend
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 80
    ```

```
kubectl apply -f my-deployment.yml
```


## Step 2: Pause the Deployment Rollout

To pause a deployment rollout, you can use the following command:

```
# Command template
kubectl rollout pause deployment <deployment-name>
{OR}
kubectl rollout pause deployment/<deployment-name>

# Actual command
kubectl rollout pause deployment my-deployment
{OR}
kubectl rollout pause deployment/my-deployment
```

This will prevent any new replicas from being created and stop the update process. You can then investigate any issues that may have occurred and make any necessary changes.



## Step 3: Update the Deployment

Let's update the deployment to use a new image `reyanshkharga/nginx:v2`.

In the Deployment YAML manifest change the value of image to `reyanshkharga/nginx:v2`.

Now, apply the manifest again:

```
kubectl apply -f my-deployment.yml
```

List Pods to verify if rollout is paused:

```
kubectl get pods | grep my-deployment
```

You'll see that the rollout didn't take place because the deployment rollout is paused.



## Step 4: Resume the Deployment Rollout

To resume a deployment rollout, you can use the following command:

```
# Command template
kubectl rollout resume deployment <deployment-name>
{OR}
kubectl rollout resume deployment/<deployment-name>

# Actual command
kubectl rollout resume deployment my-deployment
{OR}
kubectl rollout resume deployment/my-deployment
```

This will continue the rollout from where it left off.

List pods to verify if rollout has resumed:

```
kubectl get pods | grep my-deployment
```

You'll see that the rollout resumes from where it left off because the deployment rollout is resumed.



## Clean Up

Delete the deployment:

```
kubectl delete -f my-deployment.yml
```


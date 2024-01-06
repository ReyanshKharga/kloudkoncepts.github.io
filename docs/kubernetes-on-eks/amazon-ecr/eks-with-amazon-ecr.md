---
description: Unlock the power of EKS and Amazon ECR for seamless container management. Discover expert insights, tips, and best practices for leveraging EKS with Amazon ECR for your containerized applications.
---

# EKS With Amazon ECR

You can use Amazon ECR with EKS provided the worker nodes have the required permissions.

At minimum, the IAM role must have the following permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetAuthorizationToken"
            ],
            "Resource": "*"
        }
    ]
}
```


In our case, the worker nodes already have these permissions. Remember the `imageBuilder` IAM add-on policies that we used when we created the EKS cluster using `eksctl`.

The `imageBuilder` policy allows for full ECR (Elastic Container Registry) access.

Now, let's see how you can use images from ECR in your EKS Deployments.


## Step 1: Create Kubernetes Deployment

Let's create a kubernetes deployment that uses the image from ECR.

=== ":octicons-file-code-16: `deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: <account-id>.dkr.ecr.<aws-region>.amazonaws.com/my-nginx-repository:latest
            resources:
              requests:
                cpu: "1"
    ```

Ensure to update the value of `image` field in `.spec.containers` with the image URI linked to your account and region.

Apply the manifest to create the deployment:

```
kubectl apply -f deployment.yml
```


## Step 2: View the Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: View the HTML File Served by Nginx

```
# Start a shell session inside the nginx container
kubectl exec -it <pod-name> -- bash

# Hit the endpoint
curl localhost
```

You can also use `kubectl port-forward` to access the endpoint from your local host machine:

```
# forward port 80 of the container to port 3000 on local host machine
kubectl port-forward deploy/my-deployment 3000:80
```

Open any browser on your local host machine and hit localhost:3000.



## Clean Up

Let's delete the Deployment we created:

```
kubectl delete -f deployment.yml
```
---
description: Explore the world of Persistent Volumes with our hands-on demo using local storage. Learn how to effectively manage data in Kubernetes.
---

# Persistent Volume Demo Using Local Storage

Let's see how we can create a `Persistent Volume` from a local storage using the `hostPath` provisioner.


## Step 1: Verify Directory on Worker Nodes

We'll use `/mnt/nginx` directory on worker nodes for `Persistent Volume`.

Verify that `/mnt/nginx` directory doesn't exist on any worker node:

```
# Change directory to /mnt
cd /mnt

# List contents of the directory
ls
```

You'll find that there is no directory named `nginx` in `/mnt` directory on worker nodes.


## Step 2: Create a Persistent Volume

Let's create a Persistent Volume from local storage using `hostPath` as follows:

=== ":octicons-file-code-16: `my-pv.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: my-pv
    spec:
      capacity:
        storage: 5Gi
      persistentVolumeReclaimPolicy: Retain
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/mnt/nginx"
    ```

Observe the following:

- The reclaim policy is set to `Retain`
- The access mode is set to `ReadWriteOnce`
- The PV uses the `hostPath` provisioner

Create Persistent Volume:

```
# Create PV
kubectl apply -f my-pv.yml
```

List Persistent Volumes:

```
kubectl get pv
{OR}
kubectl get persistentvolume
{OR}
kubectl get persistentvolumes
```

Describe a persistent Volume:

```
kubectl describe pv <pv-name>
{OR}
kubectl describe persistentvolume <pv-name>
{OR}
kubectl describe persistentvolumes <pv-name>
```

!!! note
    The default `Reclaim Policy` for Persistent Volume is `Retain`.


## Step 3: Create a Persistent Volume Claim

Now, let's create a PVC to request storage needed for the pod we'll create in next step:

=== ":octicons-file-code-16: `my-pvc.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-pvc
    spec:
      storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
      volumeName: my-pv # Optional. If not set the PVC will bind to a PV that satisfies the PVC resource requirements
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ```

Create Persistent Volume Claim:

```
# Create PVC
kubectl apply -f my-pvc.yml
```

The default `Storage Class` will be used if you don't explicitly set `storageClassName` to `""`. We don't want that since we are not using dynamic provisioning.

A default storage class is automatically created when you create an Amazon EKS cluster. You'll notice that it is marked `default`.

View the default storage class:

```
kubectl get sc
{OR}
kubectl get storageclass
{OR}
kubectl get storageclasses
```

You'll see the following output:

```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  30d
```

Also, all PVCs that have `storageClassName` set to `""` can be bound only to PVs that have `storageClassName` also set to `""`.

List Persistent Volume Claims:

```
kubectl get pvc
{OR}
kubectl get persistentvolumeclaim
{OR}
kubectl get persistentvolumeclaims
```

Describe a persistent Volume Claim:

```
kubectl describe pvc <pvc-name>
{OR}
kubectl describe persistentvolumeclaim <pv-name>
{OR}
kubectl describe persistentvolumeclaims <pv-name>
```

Observe the following:

1. The status of `my-pvc` is `Bound`
2. The Persistent Volume Claim (PVC) `my-pvc` is bound to `my-pv` Persistent Volume (PV)


## Step 4: Create Pods That Uses the Persistent Volume Claim

Let's create pods that uses the Persistent Volume Claim we created in the previous step. We'll use a deployment to create pods:

!!! note
    While it is possible for multiple pods to utilize the same PVC, the practical implementation can be more intricate. When multiple pods need to access a `Persistent Volume` mounted with a `ReadWriteOnce` access mode, they must be scheduled on the same node to enable simultaneous access to the volume.

To keep it simple we'll create deployment with only 1 replica pod as follows:


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
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx
              ports:
                - containerPort: 80
                  name: "http-server"
              volumeMounts:
                - mountPath: "/usr/share/nginx/html"
                  name: my-volume
          volumes:
            - name: my-volume
              persistentVolumeClaim:
                claimName: my-pvc
    ```

Obeserve the following:

- The deployment creates pods with 1 replica
- Each pod has one container named `nginx`
- A volume named `my-volume` is created from the `Persistent Volume Claim`
- The volume `my-volume` is mounted on `/usr/share/nginx/html` directory of the `nginx` container


Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 5: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods -o wide
```

## Step 6: View the Page Served By Nginx Container

1. Open a shell session inside the `nginx` container of one of the pods:

    ```
    kubectl exec -it <pod-name> -- bash
    ```

2. Get the nginx page:

    ```
    curl localhost
    ```

You'll receive `403` error page because there is nothing at `/usr/share/nginx/html`.

Usually there is a default `index.html` file at `/usr/share/nginx/html` but since we mounted the local storage on worker node to `/usr/share/nginx/html`, the content of `/usr/share/nginx/html` in the `nginx` container is overwritten.

In Summary, whatever is present on `/mnt/nginx` on worker node, the same will be available to `nginx` container on `/usr/share/nginx/html`.



## Step 7: Add the Nginx Page

Connect to the worker node where the pod is running using SSH or session manager.

You'll see the directory `/mnt/nginx` is created as soon as the pod comes up.

Create a file called `index.html` with the content below at `/mnt/nginx`:

=== ":octicons-file-code-16: `index.html`"

    ```html linenums="1"
    <!DOCTYPE html>
    <html>
    <body>

    <h1>Hello from nginx pod</h1>
    <p>The pod uses a persistent volume</p>

    </body>
    </html>
    ```

Access the nginx page again:

```
# Open a shell session inside the same nginx container
kubectl exec -it <pod-name> -- bash

# Get the nginx page
curl localhost
```

You'll observe that the nginx serves the html page we created.


## Step 8: Delete the Deployment and Persistent Volume Claim (PVC)

You need to delete the deployment before you can delete the PVC because pods uses the claim as volume.

1. Delete the deployment:

    ```
    kubectl delete -f my-deployment.yml
    ```

2. Delete the PVC:

    ```
    kubectl delete -f my-pvc.yml
    ```


## Step 9: Verify the Status of Persistent Volume (PV)

List PVs:

```
kubectl get pv
```

You'll observe that the status of PV is `Released` because the claim bound to this PV has been deleted.


## Step 10: Delete the Persistent Volume (PV)

Delete the PV we created:

```
kubectl delete -f my-pv.yml
```

## Step 11: Verify That Data is Retained

Since the reclaim policy is set to the default value `Retain`, the data on worker node will be retained.

You can verify it by logging into the worker node.
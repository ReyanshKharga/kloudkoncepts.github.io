---
description: Master the art of static provisioning for Persistent Volumes using EBS with our comprehensive guide. Learn how to configure and manage storage resources in Kubernetes proactively.
---

# Static Provisioning of Persistent Volume Using EBS

Let's see how we can use EBS CSI Driver to create a Persistent Volume from an existing Amazon EBS volume.


## Step 1: Create an Amazon EBS Volume

Create an Amazon EBS volume:

```
aws ec2 create-volume \
    --volume-type gp3 \
    --size 5 \
    --encrypted \
    --availability-zone ap-south-1a \
    --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=my-ebs-volume}]'
```

Make sure you specify the `availability-zone` where at least one kubernetes worker node is running. This is because this storage will eventually be mounted on a worker node.

If the command executes successfully, you should see an output similar to below:

```json
{
    "AvailabilityZone": "ap-south-1a",
    "CreateTime": "2023-04-21T08:22:12+00:00",
    "Encrypted": true,
    "Size": 5,
    "SnapshotId": "",
    "State": "creating",
    "VolumeId": "vol-0acc3dba885b0f700",
    "Iops": 3000,
    "Tags": [
        {
            "Key": "Name",
            "Value": "my-ebs-volume"
        }
    ],
    "VolumeType": "gp3",
    "MultiAttachEnabled": false,
    "Throughput": 125
}
```

Note down the `VolumeId` from the output. We'll need it when we create Persistent Volume from this EBS volume.

Also, go to AWS console and verify the EBS volume we just created.

The volume state should be `Available` which means that the volume is not currently attached to any instance and is therefore not in use.



## Step 2: Create Persistent Volume (PV)

Let's create a Persistent Volume using the Amazon EBS volume we created in the first step:

=== ":octicons-file-code-16: `my-pv.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: my-pv
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 5Gi
      persistentVolumeReclaimPolicy: Retain
      csi:
        driver: ebs.csi.aws.com
        fsType: ext4
        volumeHandle: vol-0acc3dba885b0f700
    ```

Replace the value of `volumeHandle` with the volume id that you recorded in Step 1.


Observe the following:

- The reclaim policy is set to `Retain`
- The access mode is set to `ReadWriteOnce`
- The PV uses the `csi` provisioner
- The provisioner uses the `EBS CSI driver` to create PV using Amazon EBS volume
- The volume will be formatted and mounted using the `ext4` file system.

Create persistent volume:

```
kubectl apply -f my-pv.yml
```

Verify the status of persistent volume:

```
kubectl get pv
```

You'll observe that the status of the Persistent Volume `my-pv` is `Available` and it is not bound to any claim.



## Step 3: Create Persistent Volume Claim (PVC)

Let's create a Persistent Volume Claim as follows:

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


Observe the following:

- The `storageClassName` is set to `""` because we are not using dyanic provisioning.
- The access mode is set to `ReadWriteOnce`. This means the PVC can only be bound to PVs that have the access mode set to `ReadWriteOnce`.
- The PVC requests `5Gi` storage. This means the PVC can only be bound to PVs that have at least `5Gi` storage available.
- We have explicitly specified the name of the PV to which the PVC should be bound. If we omit the `volumeName` field, the PVC will bind to a PV that satisfies the PVC's resource requirements, such as the amount of storage needed and the required access mode.

Create persistent volume claim:

```
kubectl apply -f my-pvc.yml
```

Verify the status of persistent volume claim:

```
kubectl get pvc
```

You'll observe that the PVC `my-pvc` is bound to the PV `my-pv`.


Verify the status of persistent volume again:

```
kubectl get pv
```

You'll notice that the status of the PV `my-pv` changes from `Available` to `Bound`.

The Amazon EBS volume state will still be `Available`. The status will change to `in-use` only when the EBS volume is mounted on a worker node.


## Step 4: Create Pods That Uses the Persistent Volume Claim

Let's create pods that uses the Persistent Volume Claim we created in the previous step. We'll use a deployment to create pods.


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
            image: reyanshkharga/nginx:v1
            imagePullPolicy: Always
            command: ["/bin/sh"]
            args: ["-c", "while true; do echo $(date -u) >> /my-data/my-persistent-data.txt; sleep 5; done"]
            volumeMounts:
            - name: my-volume
              mountPath: /my-data
          volumes:
          - name: my-volume
            persistentVolumeClaim:
              claimName: my-pvc
    ```

Obeserve the following:

- The deployment creates pods with 1 replica
- The pod has one container named `nginx`
- A volume named `my-volume` is created using the PVC `my-pvc` we created in the previous step
- The volume `my-volume` is mounted on `/my-data` directory of the `nginx` container

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```

Verify the EBS volume state in AWS console. You'll observe that as soon as the pod is scheduled on a node, the EBS volume is mounted on that node and the state changes from `Available` to `in-use`.

You can see the `Attached instances` field in the EBS details section in the AWS console.


!!! note "Important Note"

    In the deployment described above, if the number of replicas is set to a value greater than one, only one pod will be created and scheduled due to the limitations of EBS volumes, which cannot be mounted on multiple nodes and, therefore, multiple pods.

    This limitation can be overcome by using `StatefulSets`, which allow for dynamic provisioning of storage for each pod in the deployment, ensuring that each pod has its own unique storage that persists even if the pod is terminated or rescheduled.


## Step 5: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods -o wide
```


## Step 6: Verify Volume Mount and Data

1. Open a shell session inside the nginx container:

    ```
    kubectl exec -it <pod-name> -- bash
    ```

2. View data:

    ```
    tail -f /my-data/my-persistent-data.txt
    ```

## Step 7: Delete the Deployment and Persistent Volume Claim (PVC)

You need to delete the deployment before you can delete the PVC because Pods uses the claim as volume.

1. Delete the deployment:

    ```
    kubectl delete -f my-deployment.yml
    ```

2. Delete the PVC:

    ```
    kubectl delete -f my-pvc.yml
    ```

## Step 8: Verify the Status of Persistent Volume (PV)

List PVs:

```
kubectl get pv
```

You'll observe that the status of PV is `Released` because the claim bound to this PV has been deleted.



## Step 9: Delete the Persistent Volume (PV)

Delete the PV:

```
kubectl delete -f my-pv.yml
```

The EBS volume will not be deleted even if you set the reclaim policy to `Delete`. This is because the EBS volume was not provisioned dynamically and, therefore, is not managed by any provisioner or driver.

Also, verify the EBS volume state in AWS console. You'll observe that as soon as the Persistent Volume is deleted, the EBS volume state changes from `in-use` to `Available`.


## Step 10: Clean Up


Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-pv.yml
│   |-- my-pvc.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

Also, delete the EBS volume:

```
aws ec2 delete-volume --region <region-name> --volume-id <volume-id>
```

Go to AWS console and verify if the EBS volume was deleted.
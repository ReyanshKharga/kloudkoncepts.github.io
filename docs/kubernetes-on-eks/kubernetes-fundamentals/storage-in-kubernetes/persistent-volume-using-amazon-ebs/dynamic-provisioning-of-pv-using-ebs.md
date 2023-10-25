---
description: Explore the dynamic provisioning of Persistent Volumes with EBS in Kubernetes. Learn how to automatically configure and manage storage resources with our comprehensive guide. Start leveraging the power of dynamic provisioning using EBS!
---

# Dynamic Provisioning of Persistent Volume Using EBS

Let's create a `StorageClass` that specifies the properties of the storage that will be dynamically provisioned for a Persistent Volume.

=== ":octicons-file-code-16: `my-sc.yml`"

    ```yaml linenums="1"
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: my-sc
    provisioner: ebs.csi.aws.com
    volumeBindingMode: Immediate
    parameters:
      type: gp3
      tagSpecification_1: "Name=my-ebs-volume"
      tagSpecification_2: "CreatedBy=aws-ebs-csi-driver"
    reclaimPolicy: Delete
    ```

Observe the following:

- The `EBS CSI Driver` is used as the provisioner to provision EBS volume dynamically
- The `parameters` field specifies the properties of the storage provisioned
- We have set the `reclaimPolicy` to `Delete`. This means that the EBS volume will be deleted when the associated Persistent Volume is deleted.
- The `tagSpecification_<i>` is used to specify the tags of EBS volume that will be provisioned

For the `ebs.csi.aws.com` (EBS CSI Driver) storage provisioner, the following are some of the additional fields that can be added to the parameters field:

1. `fsType`: The file system type to use for the volume. For example, "ext4" or "xfs".
2. `encrypted`: A boolean value that indicates whether to create an encrypted volume. Default is "false".
3. `kmsKeyId`: The ID of the KMS key to use for encryption. This field is only used if encrypted is set to "true".
4. `iopsPerGB`: The number of IOPS per GiB for the volume. This field is only used for specific volume types, such as `io1`.
5. `throughput`: The throughput in MiB/s for the volume. This field is only used for specific volume types, such as `gp3`.



---
description: Learn the basics of the EBS CSI Driver with our simple introduction. Understand its significance and how it can enhance your storage in Kubernetes.
---

# Introduction to EBS CSI Driver

Before we start learning about the `EBS CSI Driver`, let's first get a basic understanding of what the Container Storage Interface (CSI) is. This will make it easier to understand the `EBS CSI driver`.


## What is Container Storage Interface (CSI)?

Container Storage Interface (CSI) is an industry standard interface for external storage systems to be used with container orchestrators like kubernetes.

It provides a way for storage vendors to develop and deploy their own plugins that can be integrated with kubernetes to provide storage capabilities to containers.

The CSI specification defines a set of interfaces that storage vendors can implement to allow kubernetes to interact with their storage systems. This allows kubernetes to dynamically provision and manage storage resources for containers running in the cluster.

With CSI, kubernetes users can now choose from a wide variety of storage options from different vendors, and seamlessly integrate them into their kubernetes clusters.


## What is EBS CSI Driver?

Amazon Elastic Block Store (EBS) Container Storage Interface (CSI) Driver is a plugin for kubernetes that allows you to use EBS volumes as persistent volumes in kubernetes.

The `EBS CSI Driver` provides a seamless integration between EBS and Kubernetes, allowing you to easily create, attach, and mount EBS volumes to your Kubernetes pods.

The `EBS CSI Driver` supports a variety of EBS volume types, including General Purpose SSD (gp2, gp3), Provisioned IOPS SSD (io1), Throughput Optimized HDD (st1), and Cold HDD (sc1). It also provides support for snapshotting and cloning EBS volumes.

Using the `EBS CSI Driver`, you can manage your EBS volumes and Kubernetes pods independently, while still allowing them to interact seamlessly. This makes it easy to create and manage stateful applications in Kubernetes using EBS as the underlying storage.
---
description: Explore the power of Cluster Autoscaling in Amazon EKS with our introductory guide. Learn how to dynamically scale your Kubernetes clusters to meet changing demand effortlessly. Start optimizing your EKS infrastructure with Cluster Autoscaling.
---

# Introduction to Cluster Autoscaling in EKS

Autoscaling is a function that automatically scales your resources up or down to meet changing demands. This is a major kubernetes function that would otherwise require extensive human resources to perform manually.

In this course, we'll mainly talk about [Cluster Autoscaler]{:target="_blank"}, a well-known tool for adjusting the size of EKS clusters based on workload demands. While [Karpenter]{:target="_blank"} is another useful autoscaling tool, we'll cover it as a bonus topic towards the end of the course.

The kubernetes `Cluster Autoscaler` automatically adjusts the number of nodes in your cluster when pods fail or are rescheduled onto other nodes.

The primary goal of the `Cluster Autoscaler` is to ensure that there are enough resources available to meet the demand of the applications running in the cluster, and to scale the cluster size up or down accordingly.


## How Does Cluster Autoscaler Work?

Here's how Cluster Autoscaler works in kubernetes:

<p align="center">
    <img src="../../../../assets/eks-course-images/autoscaling/working-of-cluster-autoscaler.png" alt="Working of Cluster Autoscaler" width="550" />
</p>

1. Kubernetes Cluster Autoscaler detects pods in the `pending` state due to insufficient resources.
2. The Cluster Autoscaler increases the desired number of instances in the Autoscaling Group.
3. AWS Autoscaling provisions new nodes to match the desired instances count.
4. Finally, the pending pods are scheduled on the new nodes that come up.




!!! quote "References:"
    !!! quote ""
        * [Cluster Autoscaler]{:target="_blank"}
        * [Karpenter]{:target="_blank"}


<!-- Hyperlinks -->
[Cluster Autoscaler]: https://aws.github.io/aws-eks-best-practices/cluster-autoscaling/
[Karpenter]: https://aws.github.io/aws-eks-best-practices/karpenter/
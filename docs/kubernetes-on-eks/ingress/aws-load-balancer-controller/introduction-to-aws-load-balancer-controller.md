---
description: Discover the power of AWS Load Balancer Controller with our comprehensive introduction. Learn how to efficiently manage and optimize load balancing in your EKS infrastructure. Get started with our informative guide and enhance your Kubernetes capabilities now!
---

# Introduction to AWS Load Balancer Controller

AWS Load Balancer Controller (LBC) is an Ingress controller to help manage AWS Elastic Load Balancers for a kubernetes cluster.

The `AWS Load Balancer Controller` automates the creation and configuration of the load balancers, ensuring that they match the desired state specified in kubernetes resources such as Services and Ingresses.

!!! note
    `AWS Load Balancer Controller` was formerly known as `AWS ALB Ingress Controller`.


## How Does AWS Load Balancer Controller Work?

The following diagram details the AWS components the AWS Load Balancer Controller creates. It also demonstrates the route ingress traffic takes from the ALB to the kubernetes cluster.

<p align="center">
    <img src="../../../../assets/eks-course-images/ingress/aws-load-balancer-controller-design.png" alt="AWS Load Balancer Controller Design" />
</p>

This section describes each step (circle) above. This example demonstrates satisfying 1 ingress resource.

1. The AWS Load Balancer Controller watches for ingress events from the API server. When it finds ingress resources that satisfy its requirements, it begins the creation of AWS resources.

2. An ALB (ELBv2) is created in AWS for the new ingress resource. This ALB can be internet-facing or internal. You can also specify the subnets it's created in using annotations.

3. Target Groups are created in AWS for each unique kubernetes service described in the ingress resource.

4. Listeners are created for every port detailed in your ingress resource annotations. When no port is specified, sensible defaults (80 or 443) are used. Certificates may also be attached via annotations.

5. Rules are created for each path specified in your ingress resource. This ensures traffic to a specific path is routed to the correct kubernetes Service.

Along with the above, the controller also:

- Deletes AWS components when ingress resources are removed from k8s.
- Modifies AWS components when ingress resources change in k8s.
- Assembles a list of existing ingress-related AWS components on start-up, allowing you to recover if the controller were to be restarted.


## Traffic Modes in AWS Load Balancer Controller

AWS Load Balancer controller supports two traffic modes:

- **Instance mode:** Ingress traffic starts at the ALB and reaches the kubernetes nodes through each service's NodePort. This means that services referenced from ingress resources must be exposed by `type:NodePort` in order to be reached by the ALB.

- **IP mode:** Ingress traffic starts at the ALB and reaches the kubernetes pods directly.

By default, Instance mode is used. Users can explicitly select the mode via `alb.ingress.kubernetes.io/target-type` annotation.


## AWS Load Balancer Controller Annotations

You can add [annotations] to kubernetes Ingress and Service objects to customize their behaviour.

Annotations in the AWS Load Balancer Controller are additional metadata or configuration settings that can be added to kubernetes resources (such as Services and Ingresses) to provide specific instructions or customizations for the load balancer configuration.

By adding annotations to kubernetes resources, you can configure advanced features like SSL/TLS termination, sticky sessions, health checks, target group attributes, cross-zone load balancing, access logs, security policies, and more.



!!! quote "References:"
    !!! quote ""
        * [AWS Load Balancer Controller]{:target="_blank"}
        * [Ingress Annotations - AWS Load Balancer Controller]{:target="_blank"}


<!-- Hyperlinks -->
[annotations]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/
[AWS Load Balancer Controller]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/
[Ingress Annotations - AWS Load Balancer Controller]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/guide/ingress/annotations/
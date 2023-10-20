---
description: Understand Kubernetes Service with AWS Application Load Balancer (ALB). Discover why creating an ALB in AWS directly from a Kubernetes Service requires specific controllers or Ingress.
---

# Kubernetes Service With AWS Application Load Balancer (ALB)

It is not possible to create an Application Load Balancer (ALB) in AWS directly from a kubernetes service without using any controller or `Ingress`.

Kubernetes service type `LoadBalancer` only creates a `NodePort` service, and it relies on cloud-specific integrations (like controllers) to create and manage the external load balancers in the cloud provider.

Without any controller or Ingress, the kubernetes cluster has no knowledge of how to provision and manage the ALB in AWS. So, the kubernetes service type `LoadBalancer` alone cannot create an ALB in AWS.

We'll learn how to use `AWS Load Balancer Controller` to create kubernetes service with ALB later in this course. Stay tuned!

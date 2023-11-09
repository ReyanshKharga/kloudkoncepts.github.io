---
description: Easily Offload NLB Reconciliation to AWS Load Balancer Controller with loadBalancerClass. Simplify management and optimize your Network Load Balancer (NLB) with LBC. 
---

# Offload Reconciliation of NLB to LBC Using loadBalancerClass

In this demo, we will use `spec.loadBalancerClass` to offload the reconciliation of Network Load Balancer (NLB) to AWS Load Balancer Controller (LBC).

!!! note
    If you configure `spec.loadBalancerClass`, the `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` defaults to `instance`.
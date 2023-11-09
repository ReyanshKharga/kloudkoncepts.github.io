---
description: Simplify NLB management in Kubernetes with AWS Load Balancer Controller. Offload reconciliation and boost your Network Load Balancer (NLB) using AWS Load Balancer Controller.
---

# Introduction to NLB using AWS Load Balancer Controller

By default, kubernetes service resources of type `LoadBalancer` gets reconciled by the kubernetes controller built into the CloudProvider component of the `kube-controller-manager` or the `cloud-controller-manager` (a.k.a. the `in-tree controller`).

In order to let AWS Load Balancer Controller (LBC) manage the reconciliation for kubernetes services resources of type `LoadBalancer`, you need to offload the reconciliation from `in-tree controller` to AWS Load Balancer Controller explicitly.

You can offload the reconciliation to AWS Load Balancer Controller in two ways:

1. By specifying the `spec.loadBalancerClass` and set it to `service.k8s.aws/nlb`
2. By specifying the `service.beta.kubernetes.io/aws-load-balancer-type` annotation and set it to `external` or `nlb-ip`

The AWS Load Balancer Controller manages kubernetes services in a compatible way with the legacy aws cloud provider. The annotation `service.beta.kubernetes.io/aws-load-balancer-type` or `spec.loadBalancerClass` is used to determine which controller reconciles the service.

If `spec.loadBalancerClass` is set or the annotation value is `nlb-ip` or `external`, legacy cloud provider ignores the service resource (provided it has the correct patch) so that the AWS Load Balancer controller can take over. For all other values of the annotation, the legacy cloud provider will handle the service.

!!! note

    The annotation `service.beta.kubernetes.io/aws-load-balancer-type` should be specified during service creation and not edited later. The value `nlb-ip` is deprecated and might be removed later. Use the value `external` instead.


## NLB Target Type

You must provide `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` annotation if you are using `service.beta.kubernetes.io/aws-load-balancer-type` annotation to offload the reconciliation to AWS Load Balancer Controller.

If you configure `spec.loadBalancerClass`, the `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` defaults to `instance`.


!!! quote "References:"
    !!! quote ""
        * [NLB Using Load Balancer Controller]{:target="_blank"}
        * [NLB Annotations]{:target="_blank"}


<!-- Hyperlinks -->
[NLB Using Load Balancer Controller]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[NLB Annotations]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/service/annotations/
---
description: Understand how ExternalDNS simplifies Kubernetes resource discovery with public DNS servers. Unlike Kubernetes internal DNS server, ExternalDNS configures other DNS providers like AWS Route 53 and Google Cloud DNS. Learn how it helps you make Kubernetes resources accessible via public DNS.
---

# Introduction to ExternalDNS

In the previous sections, we had to manualy map a subdomain to a load balancer. Kubernetes `ExternalDNS` automates the creation, updation, and deletion of the Route 53 records.

`ExternalDNS` is a kubernetes tool that draws inspiration from kubernetes DNS and enhances resource discoverability through public DNS servers.

Unlike kubernetes' internal DNS server, KubeDNS, `ExternalDNS` does not act as a DNS server in itself. Instead, it leverages the kubernetes API to gather a comprehensive list of resources, such as Services and Ingresses, and then configures external DNS providers, like AWS Route 53 or Google Cloud DNS, to create the desired DNS records.

This functionality allows kubernetes resources to become readily accessible via public DNS servers, offering greater flexibility and integration for managing domain names in kubernetes environments.

`ExternalDNS` creates DNS records based on the host information. `ExternalDNS` sets up and manages records in Route 53 that point to controller deployed ALBs.

For ingress objects `ExternalDNS` will create a DNS record based on the `hosts` specified for the ingress object, as well as the `external-dns.alpha.kubernetes.io/hostname` annotation.

For services `ExternalDNS` will look for the annotation `external-dns.alpha.kubernetes.io/hostname` on the service and use the loadbalancer DNS to create Route 53 record.



!!! quote "References:"
    !!! quote ""
        * [External DNS]{:target="_blank"}


<!-- Hyperlinks -->
[External DNS]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/integrations/external_dns/
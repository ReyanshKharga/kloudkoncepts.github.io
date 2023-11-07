---
description: Secure your applications and enhance user trust. Learn how to create Kubernetes Ingress with SSL in our comprehensive guide. Discover the essential steps to enable SSL encryption for your services and protect sensitive data. Elevate your Kubernetes deployment with SSL-enabled Ingress.
---

# Create Ingress With SSL

SSL support can be controlled with following annotations:

| Annotation | Function |
|----------------------------------------------|----------|
| <a>`alb.ingress.kubernetes.io/certificate-arn`</a> |specifies the `ARN` of one or more certificate managed by AWS Certificate Manager. The first certificate in the list will be added as default certificate. And remaining certificate will be added to the optional certificate list. |
| <a>`alb.ingress.kubernetes.io/ssl-policy`</a> | specifies the Security Policy that should be assigned to the ALB, allowing you to control the protocol and ciphers. This is optional and defaults to `ELBSecurityPolicy-2016-08` |


## Prerequisite

To follow this tutorial, you'll require a domain and, additionally, an SSL certificate for the domain and its subdomains.

1. Register a Route53 Domain

    Go to AWS Console and register a Route53 Domain. You can opt for a cheaper TLD (top level domain) such as `.link`

    !!! note
        It usually takes about 10 minutes but it might take about an hour for the registered domain to become available.

2. Request a Public Certificate

    Visit AWS Certificate Manager in AWS Console and request a public certificate for your domain and all the subdomains. For example, if you registered for a domain `xyz.link` then request certificate for `xyz.link` and `*.xyz.link`

    !!! note
        Make sure you request the certificate in the region where your EKS cluster is in.

3. Validate the Certificate

    Validate the requested certificate by adding `CNAME` records in Route53. It is a very simple process. Go to the certificate you created and click on `Create records in Route53`. The CNAMEs will be automatically added to Route53.

    !!! note
        It usually takes about 5 minutes but it might take about an hour for the certificate to be ready for use.


Now that you have everything you need, let's move on to the demonstration.



## Step 

---
description: Explore the fundamentals of Fault Injection in our comprehensive guide! Learn how to uncover vulnerabilities, enhance system resilience, and optimize performance through strategic fault introduction.
---

# Introduction to Fault Injection

Fault injection testing is a software testing method where the errors and delays are deliberately introduced in a system to check and ensure that the system can withstand and recover from error conditions.

You may want to test how delayed response or faults in a service affects your applications. You don't want to experience it in your production environment and therefore you can test your applications with fault injection technique before it goes live.

Istio allows you to implement fault injection to test the resiliency of your application. With this feature, you can use application-layer fault injection instead of killing pods, delaying packets, or corrupting packets at the TCP layer.

The Fault injection is handled by the VirtualService Istio Object.



## Types of Fault Injection

Istio defines two types of faults injection:

- `Delays`: Delays are timing failures such us network latency or overloaded upstreams.
- `Aborts`: Aborts are crash failures such as HTTP error codes or TCP connection failures.




!!! quote "References:"
    !!! quote ""
        * [Fault Injection]{:target="_blank"}


<!-- Hyperlinks -->
[Fault Injection]: https://www.istioworkshop.io/09-traffic-management/05-fault-injection/
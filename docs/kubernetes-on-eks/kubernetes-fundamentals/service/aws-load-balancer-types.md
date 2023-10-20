---
description: Explore AWS Load Balancer Types. Learn about the various load balancer options offered by AWS, including their features and use cases. Make informed decisions for optimizing your application's performance and availability on the AWS cloud.
---

# AWS Load Balancer Types

Amazon Web Services (AWS) offers several types of load balancers that can help distribute incoming traffic across multiple instances or targets, increasing availability and fault tolerance.

The main types of load balancers offered by AWS are:

1. **Network Load Balancer (NLB):**

    This load balancer operates at the transport layer (Layer 4) and is optimized for handling high volumes of traffic. It can handle millions of requests per second while maintaining ultra-low latency. 
    
    It is typically used for TCP and UDP traffic, and supports static IP addresses, which can be used for whitelisting and more complex routing scenarios.

2. **Classic Load Balancer (CLB):**

    This load balancer is the original load balancer offered by AWS, and operates at both the application and transport layers (Layer 7 and Layer 4). 
    
    It provides basic load balancing capabilities and supports HTTP, HTTPS, TCP, and SSL protocols. However, it does not offer the advanced features and flexibility of the ALB or NLB.

3. **Application Load Balancer (ALB):**

    This load balancer operates at the application layer (Layer 7) and provides advanced routing capabilities, including content-based routing, support for HTTP/2, WebSocket, and integration with AWS services such as AWS Certificate Manager, AWS WAF, and AWS Elastic Container Service (ECS). 
    
    It can also route traffic to multiple targets within a single Availability Zone or across multiple Availability Zones.
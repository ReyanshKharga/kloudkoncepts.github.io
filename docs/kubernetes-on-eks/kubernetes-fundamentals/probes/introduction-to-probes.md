---
description: Get started with Kubernetes Probes in our comprehensive introduction. Understand how probes improve container health and reliability.
---

# Introduction to Probes in Kubernetes

Let's say you have an e-commerce application running in a container, and you want to make sure it's always available for your customers to use.

However, there's always a risk that the application could become unavailable due to a variety of reasons, such as a bug in the application, a connection error with the database, or some other issue.

This is where kubernetes probes come in. By setting up probes for your application, kubernetes can monitor its health and automatically take action if any issues arise.

For example, if a probe detects that the application is not responding, kubernetes can automatically restart the container or remove it from the pool of available instances, ensuring that the application is always available for your customers.


## What is a Probe in Kubernetes?

Kubernetes probes are health checks that are performed periodically by the kubelet on a container.

To perform a health check, the kubelet either executes code within the container, or makes a network request.

If a container fails a health check defined by a probe, kubernetes can automatically take corrective action, such as restarting the container or removing the pod from the pool of available pods.


## Health Check Mechanisms

There are three main health check mechanisms used by probes:

1. **exec:** Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of `0`.

2. **httpGet:** Performs an `HTTP GET` request against the pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to `200` and less than `400`.

3. **tcpSocket:** Performs a TCP check against the pod's IP address on a specified port. The diagnostic is considered successful if the port is open.
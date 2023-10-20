---
description: Uncover the differences between port, targetPort, and nodePort in Kubernetes Services. Demystify the essential parameters, gaining a clear understanding of how they influence service configuration in your Kubernetes environment.
---

# Difference Between port, targetPort, And nodePort in Kubernetes Service

A kubernetes service can have different types of ports fields, including `port`, `targetPort`, and `nodePort`.

Here's a brief explanation of each:

- `port`: This is the port on which the service listens. When a client sends a request to the service, it connects to this port.

- `targetPort`: This is the port on which the pods in the service are listening. When the service forwards traffic to the pods, it uses this port.

- `nodePort`: This is a port that is exposed on each node in the cluster. When traffic comes to this port, it is forwarded to the service's port.

<p align="left">
    <img src="../../../../assets/eks-course-images/service/port-vs-targetport-vs-nodeport.png" alt="Difference Between port, targetPort, And nodePort in Kubernetes Service" />
</p>

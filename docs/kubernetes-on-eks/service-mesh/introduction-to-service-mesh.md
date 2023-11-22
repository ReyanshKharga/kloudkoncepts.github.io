---
description: Explore the basics of service mesh in our comprehensive guide! Learn how it streamlines communication between microservices in a Kubernetes environment. Dive into key concepts and benefits of service mesh.
---

# Introduction to Service Mesh

A service mesh is a dedicated infrastructure layer that you can add to your applications. It allows you to transparently add capabilities like observability, traffic management, and security, without adding them to your own code.


## Life Before Service Mesh

Let's say you are building a service-based architecture consisting of multiple microservices. Any given microservice may interact with many other microservices.

For service to service communication between microservices the developers must implement the following:

1. **Retry Logic and Circuit Breaking**

    Let's say service `A` makes a call to service `B`.

    - If the call to service `B` fails, service `A` may want to retry
    - Service `A` must have retry logic in the codebase (frequency, interval etc.)
    - The retry logic may depend on the microservices involved in the picture
    - Service `A` must detect failures and encapsulate the logic of preventing a failure from constantly recurring (Circuit Breaking)

2. **Authentication and Authorization**

    Let's say service `A` makes a call to service `B`.

    - You may want authentication between service `A` and `B`
    - Service `A` and service `B` must have authentication logic in the codebase

3. **Mutual TLS**

    Let's say service `A` makes a call to service `B`.

    - You may want service `A` and service `B` to interact over `HTTPS` instead of `HTTP`
    - The certificate for each service should be issued, maintained, and rotated for each service

4. **Monitoring**

    Let's say service `A` makes a call to service `B`.

    - You may want to know the number of requests/sec the service `A` sends
    - You may also want to know the number of requests/sec the service `B` receives
    - You may want to know metrics like latency, HTTP 200, 4xx, and 5xx count
    - The developer must write logic to expose these metrics

5. **Distributed Tracing**

    Let's say service `A` makes a call to service `B` and service `B` in turn call service `C` and `D`.

    - You may want to trace requests down to each service to figure out where the latency may be
    - You may want to visualize the flow of requests between microservices

6. **Traffic Splitting**

    Let's say service `A` makes a call to service `B`. Service `B` has two versions `B1` and `B2`.

    - You may want to send only 10% of the traffic to `B1` and rest 90% to `B2`

<p align="center">
    <img src="../../../assets/eks-course-images/service-mesh/code-oriented-frameworks.png" alt="Code Oriented Framework" loading="lazy" />
</p>

There are many other features like load balancing, rate limiting, service registry/discovery that the developer should implement.

Consider implementing these features for all microservices ([Code Oriented Pattern]{:target="_blank"}). This is very painful and not an easy task. The complexity grows as more and more microservices are added. And of course, this is not scalable for following reasons:

- Since each microservice might be written in different programming languages, implementing these features for each microservice becomes language-specific.
- Prone to errors during implementation.
- Difficult to upgrade each microservice as the system grows.
- Different teams within the same organization may adopt different implementations.


That's where service mesh comes into the picture!

Service mesh provides you these features out of the box without adding any additional code in your microservices.



## Sidecar Pattern

Sidecar pattern enables applications to be composed of heterogeneous components and technologies. This pattern is named Sidecar because it resembles a sidecar attached to a motorcycle.

In the pattern, the sidecar is attached to a parent application and provides supporting features for the application. Thus, all microservices concerns will be handled by the sidecar in homogenous way and injected on the fly by platform when deploying the application.

<p align="center">
    <img src="../../../assets/eks-course-images/service-mesh/sidecar-oriented-pattern.png" alt="Sidecar Oriented Pattern" loading="lazy" />
</p>


## What is Istio?

Istio is an implementation of service mesh for observability, security in depth, and management that speeds deployment cycles.

It is an open source service mesh that layers transparently onto existing distributed applications.



## Istio Architecture

Istio is logically splitted into a `control plane` and a `data plane`.

<p align="center">
    <img src="../../../assets/eks-course-images/service-mesh/servicemesh-highlevel-architecture.png" alt="Architecture of Service Mesh" loading="lazy" width="450" />
</p>

### Control Plane

The control plane is the brain of the main network that manages, controls, and supervises the network of microservies.

The control plane manages and configures the proxies to route traffic. Additionally, the control plane configures `Mixers` to enforce policies and collect telemetry.


### Data Plane

The data plane is composed of a set of intelligent proxies (Envoy) deployed as sidecars.

These proxies mediate and control all network communication between microservices along with `Mixer`, a general-purpose policy and telemetry hub.

The sidecars deployed within the services and acting as proxy form the service mesh network.


## Building Blocks of Istio

<p align="center">
    <img src="../../../assets/eks-course-images/service-mesh/istio-components-interaction.png" alt="Istio Components Interaction" loading="lazy" />
</p>


| Component | Description | Mandatory | Group |
|-----------|-------------|-----------|-------|
| <a>`Pilot`</a> | Responsible for service discovery and configuring envoy sidecar proxies | :octicons-check-16: | Control plane |
| <a>`Galley`</a> | Configuration ingestion for istio components | :octicons-x-16: | Control plane |
| <a>`Sidecar injector`</a> | Inside envoy sidecar for enabled namespaces | :octicons-x-16: | Control plane |
| <a>`Citadel`</a> | Automated key and certificate management | :octicons-x-16: | Control plane |
| <a>`Policy`</a> | Policy enforcement | :octicons-x-16: | Control plane |
| <a>`Telemetry`</a> | Gather telemetry data | :octicons-x-16: | Control plane |
| <a>`Ingresss Gateway`</a> | Manage inbound connection to the service mesh | :octicons-x-16: | Control plane |
| <a>`Egress Gateway`</a> | Manage outbound connection from the service mesh | :octicons-x-16: | Control plane |
| <a>`Istio CNI`</a> | Network initialisation | :octicons-x-16: | Control plane |
| <a>`Prometheus`</a> | Metrics collections | :octicons-x-16: | Control plane |
| <a>`Core DNS`</a> | DNS resolution in a multicluster gateways deployment | :octicons-x-16: | Control plane |
| <a>`Cert Manager`</a> | Issuance and renewal of TLS certificates | :octicons-x-16: | Control plane |
| <a>`Grafana`</a> | Grafana	Monitoring dashboard | :octicons-x-16: | Dashboard |
| <a>`Jaeger`</a> | Distributed tracing | :octicons-x-16: | Dashboard |
| <a>`Kiali`</a> | Observability dashboard | :octicons-x-16: | Dashboard |
| <a>`Envoy proxy`</a> | Proxy injected as a sidecar | :octicons-x-16: | Data plane |


- The `ingress controller` is responsible for allowing and redirecting the inbound traffic to the services running inside the service mesh.

- The `egress controller` is responsible for allowing outbound traffic from the service mesh. If an application should connect, for example, to an external database or service, such configuration should be explicitly defined for the egress controller.

- `Pilot` and `Galley` are responsible for the mesh configuration. They pull data from Kubernetes API Server and mix it with the local configuration defined within the mesh then push the configuration to different proxies forming the mesh.

- `Citadel` push tls certificate to services enabling mutual TLS.

- `Mixer` has two roles: gather metrics from the different components and enforce policy by double checking each request. In a high level deployment scenario Telemetry and Policy check should be deployed separately.

- `Dashboards` gather metrics from the telemetry service and display it in a user friendly format.


## Upstream vs Downstream

`Upstream` connections are the service Envoy is initiating the connection to while `Downstream` connections are the client that is initiating a request through Envoy.

<p align="center">
    <img src="../../../assets/eks-course-images/service-mesh/istio-upstream-downstream.png" alt="Istio Upstream vs Downstream" loading="lazy" width="450" />
</p>



!!! quote "References:"
    !!! quote ""
        * [What is a Service Mesh?]{:target="_blank"}
        * [Istio Architecture]{:target="_blank"}


<!-- Hyperlinks -->
[Code Oriented Pattern]: https://www.istioworkshop.io/03-servicemesh-overview/introduction-service-mesh/#code-oriented-pattern
[What is a Service Mesh?]: https://istio.io/latest/about/service-mesh/#what-is-a-service-mesh
[Istio Architecture]: https://www.istioworkshop.io/03-servicemesh-overview/istio-architecture/
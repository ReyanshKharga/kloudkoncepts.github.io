---
description: Effortlessly monitor Kubernetes with Prometheus and Grafana. Master monitoring for your Kubernetes environment with our step-by-step guide.
---

# Monitoring Kubernetes Using Prometheus and Grafana

We can use Prometheus and Grafana to monitor kubernetes resources, workloads and other metrics.

Prometheus and Grafana form a powerful duo for monitoring Kubernetes. They aid in tracking resource utilization, workload performance, and diverse metrics critical for maintaining cluster health.


## What is Prometheus?

Prometheus is a monitoring solution for storing time series data like metrics. It operates by periodically scraping metrics from configured targets using a pull-based model.

The core components of Prometheus include the `Prometheus Server`, which stores time-series data, a multidimensional data model, `service discovery` for dynamic target identification, and an `Alert Manager` for alerting based on predefined rules.

Using `PromQL`, users query and analyze metrics, while exporters facilitate the exposure of specific metrics from various systems.


## What is Grafana?

Grafana is a visualization and monitoring tool, offering a user-friendly interface to create dashboards and analyze data from various sources, including Prometheus.

Its key components include versatile dashboard creation, data source integration (like Prometheus), and a rich library of plugins for extended functionality.

Grafana simplifies complex data visualization, enabling users to build insightful, customizable dashboards that display metrics from multiple sources.


## Metrics and Exporters

Kubernetes exposes some basic metrics by default through an endpoint named `/metrics`. These metrics include information about the state of the kubernetes components and resources, such as the number of running pods, CPU and memory usage, API server latency, and more. 

the `/metrics` endpoint is accessible on the kubernetes API server and provides valuable insights into the cluster's health and performance. These metrics can be utilized by monitoring tools like Prometheus to gather information and create visualizations for better cluster management.

<p align="center">
    <img src="../../../assets/eks-course-images/monitoring/prometheus-and-grafana.png" alt="Kubernetes Monitoring Using Prometheus and Grafana" width="600" />
</p>

You can also set up custom exporters in kubernetes to capture precise metrics from your applications or services. These tailored tools expand monitoring capabilities, providing detailed insights beyond the standard metrics, addressing your specific monitoring needs.


## Prerequisites

We will use `helm` to install Prometheus and Grafana monitoring tools. So, make sure `helm` is installed on your local machine.


## Step 1: Add Prometheus and Grafana Helm Repository

```
# Add prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Add grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts
```


## Step 2: Deploy Prometheus

### 1. Install Prometheus

### 2. Verify Prometheus Components

### 3. Verify EBS Volumes

### 4. Access Prometheus Server

### 5. Create Ingress for Prometheus


## Deploy Grafana


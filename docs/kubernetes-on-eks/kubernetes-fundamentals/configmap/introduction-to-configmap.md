---
description: Discover the fundamentals of ConfigMap with our straightforward introduction. Learn how to manage configuration data in Kubernetes and simplify application settings.
---

# Introduction to ConfigMap

A `ConfigMap` is kubernetes is used to store non-confidential data in key-value pairs.

Pods can consume `ConfigMaps` as environment variables, command-line arguments, or as configuration files in a volume.

A `ConfigMap` allows for decoupling of configuration data from container images and provides flexibility in managing application configurations.

A `ConfigMap` is not designed to hold large chunks of data. The data stored in a `ConfigMap` cannot exceed `1 MiB`. If you need to store settings that are larger than this limit, you may want to consider mounting a volume or use a separate database or file service.


## Use Cases of ConfigMap

1. **Environment Variables:** ConfigMaps can be used to set environment variables for containers running in a pod.

2. **Application Configuration:** ConfigMaps are frequently used to store application configuration data, such as database connection strings and API endpoints.

3. **File Mounts:** ConfigMaps can be used to mount configuration files as volumes in Kubernetes pods. This allows applications to read configuration files without having to include them in the container image.


!!! quote "References:"
    !!! quote ""
        * [ConfigMaps]{:target="_blank"}


<!-- Hyperlinks -->
[ConfigMaps]: https://kubernetes.io/docs/concepts/configuration/configmap/
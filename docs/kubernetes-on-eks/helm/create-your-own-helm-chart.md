---
description: Learn how to create your own Helm chart for Kubernetes with our step-by-step guide. Understand the structure, components, and best practices for creating and customizing Helm charts. Start deploying your applications with ease.
---

# Create Your Own Helm Chart

We'll discuss about the Chart file structure in the next section. But you can get started quickly by using the `helm create` command.

```
# Command template
helm create <chart-name>

# Example
helm create my-chart
```

A folder with the name `my-chart` will be created. You can edit it and create your own templates. For now we are not going to make any changes to that.

As you edit your chart, you can validate that it is well-formed by running `helm lint` command as follows:

```
# Command template
helm lint <chart-name>

# Example
helm lint my-chart
```


## Package the Chart for Distribution

You can use the `helm package` command to package the chart for distribution:

```
# Command template
helm package <chart-name>

# Example
helm package my-chart
```

It will create a `.tgz` file that can be distributed.



## Install the Chart

Now that you have the Chart in the `.tgz` format, you can use helm install command to install it.

```
# Command template
helm install <name> <path-to-tgz-file>

# Example
helm install my-chart ./my-chart-0.1.0.tgz
```

You can use `--namespace` or `-n` argument to deploy the resources in the desired namespace. Also, you can use `--set` argument to override the default values or you can also use the `--values` or `-f` flag, followed by the path to the YAML file.

```
helm install <name> <path-to-tgz-file> --set some.key=some.value --set some.other.key=some.other.value

{OR}

helm install <name> <path-to-tgz-file> -f myvalues.yml

{OR}

helm install <name> <path-to-tgz-file> --values myvalues.yml
```

This will create kubernetes objects defined in the chart. In our case a nginx deployment with 1 replica will be created. (Take a look at values.yaml file to find out why!!)


You can see the values that you can override using the helm show values command:

```
# Command template
helm show values <chart>

# Example
helm show values ./my-chart-0.1.0.tgz
```


## Upgrade the Chart

Let's upgrade the chart by changing the image tag to latest:

```
helm upgrade my-chart my-chart-0.1.0.tgz --set image.tag=latest
```

Start a shell session inside the container and verify that nginx is running:


## Uninstall the Chart

You can use `helm uninstall` command to uninstall a chart:

```
helm uninstall my-chart
```

Uninstalling a chart deletes all the kubernetes objects that were created as part of `helm install` command.
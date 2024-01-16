---
description: Discover how to use Helm for templating in your applications, making your code more maintainable and efficient. Explore the benefits of using Helm charts and templates to streamline your development process.
---

# Using Helm for Templating

Helm is a package manager for kubernetes that can also be used for templating kubernetes manifests.

Helm templating provides a way to create more maintainable, scalable, and repeatable kubernetes manifests. By using templates, you can reduce the complexity of managing your kubernetes resources and make it easier to customize and reuse them.


## Benefits of Helm Templating

**Reusability:** Templating allows you to define reusable templates for kubernetes manifests, making it easier to create new resources that follow a certain pattern. By using variables and templates, you can create a single template that can be used to create multiple resources with different values.

**Customization:** With templating, you can easily customize your kubernetes manifests for different environments or use cases. For example, you might have different values for development, staging, and production environments. By using templates, you can easily swap out values for different environments.

**Simplification:** Kubernetes manifests can be complex and difficult to manage, especially as your applications grow. Templating allows you to abstract away some of the complexity and manage your resources at a higher level.

**Versioning:** With Helm, you can version your templates and apply them to your kubernetes cluster. This provides an audit trail of changes made to your cluster and allows you to roll back to a previous version of your templates if needed.


Now, let's see how you can write templates for your kubernetes manifests using helm.


## Step 1: Use Helm Create Command to Initialize the Chart

Initialize the chart:

```
helm create helm-chart
```

We have named our chart `helm-chart`.


## Step 2: Delete Unnecessary Files

Delete `charts/` folder. Delete all files in `templates/` folder. (We'll write our own templates) Delete all files in the root folder except `Chart.yaml`.

The `Chart.yaml` file contains metadata about the chart, such as its name, version, and description.

Here's how `Chart.yaml` file looks like:

=== ":octicons-file-code-16: `Chart.yaml`"

    ```yaml linenums="1"
    apiVersion: v2
    name: helm-chart
    description: A Helm chart for Kubernetes

    # A chart can be either an 'application' or a 'library' chart.
    #
    # Application charts are a collection of templates that can be packaged into versioned archives
    # to be deployed.
    #
    # Library charts provide useful utilities or functions for the chart developer. They're included as
    # a dependency of application charts to inject those utilities and functions into the rendering
    # pipeline. Library charts do not define any templates and therefore cannot be deployed.
    type: application

    # This is the chart version. This version number should be incremented each time you make changes
    # to the chart and its templates, including the app version.
    # Versions are expected to follow Semantic Versioning (https://semver.org/)
    version: 0.1.0

    # This is the version number of the application being deployed. This version number should be
    # incremented each time you make changes to the application. Versions are not expected to
    # follow Semantic Versioning. They should reflect the version the application is using.
    # It is recommended to use it with quotes.
    appVersion: "1.16.0"
    ```

The `charts/` folder within the created chart directory is used to store charts that the current chart depends on. In our case we don't have any dependency on other charts and that's why we deleted it.


## Step 3: Add Templates

Let's create templates for kubernetes deployment and service in the `templates/` folder.

=== ":octicons-file-code-16: `deployment.yaml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Values.appName }}-deployment
    spec:
      replicas: {{ .Values.replicas }}
      selector:
        matchLabels:
          app: {{ .Values.appName }}
      template:
        metadata:
          labels:
            app: {{ .Values.appName }}
        spec:
          containers:
          - name: {{ .Values.appName }}
            image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
            ports:
            - containerPort: {{ .Values.port }}
    ```

=== ":octicons-file-code-16: `service.yaml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Values.appName }}-service
    spec:
      selector:
        app: {{ .Values.appName }}
      ports:
      - name: http
        port: {{ .Values.port }}
        targetPort: {{ .Values.port }}
      type: {{ .Values.serviceType }}
    ```


## Step 4: Create values.yaml File

Create `values.yaml` file for default values. Additionally, create `values.prod.yaml` for prod environment and `values.dev.yaml` for dev environment.

=== ":octicons-file-code-16: `values.yaml`"

    ```yaml linenums="1"
    replicas: 1

    appName: myapp

    port: 80

    image:
      repository: reyanshkharga/nodeapp
      tag: v2

    serviceType: ClusterIP
    ```

=== ":octicons-file-code-16: `values.prod.yaml`"

    ```yaml linenums="1"
    replicas: 2

    appName: myapp

    port: 80

    image:
      repository: reyanshkharga/nodeapp
      tag: v1

    serviceType: ClusterIP
    ```

=== ":octicons-file-code-16: `values.dev.yaml`"

    ```yaml linenums="1"
    replicas: 1

    appName: myapp

    port: 80

    image:
      repository: reyanshkharga/nodeapp
      tag: v2

    serviceType: ClusterIP
    ```

At this point your folder structure should look like following:

```
|-- helm-chart
│   |-- templates
│   |   |-- deployment.yaml
│   |   |-- service.yaml
│   |-- templates
│   |-- Chart.yaml
│   |-- values.dev.yaml
│   |-- values.prod.yaml
│   |-- values.yaml
```


## Step 5: Install Chart

Now that we have the templates/chart ready, we can install it.

```
# Install in default namespace with default values
helm install mychart helm-chart/

# Install in dev namespace
kubectl create ns dev
helm install mychart helm-chart/ --values helm-chart/values.dev.yaml --namespace dev

# Install in prod namespace
kubectl create ns prod
helm install mychart helm-chart/ --values helm-chart/values.prod.yaml --namespace prod
```


## Step 6: Verify Resources

Verify the kubernetes objects that were created as part of the helm install process.

First, let's verify the charts installed:

```
# Verify chart in default namespace
helm list

# Verify chart in dev namespace
helm list -n dev

# Verify chart in prod namespace
helm list -n prod
```

Next, let's list the kubernetes resources in each namespace:

```
# List kubernetes resources in default namespace
kubectl get all

# List kubernetes resources in dev namespace
kubectl get all -n dev

# List kubernetes resources in prod namespace
kubectl get all -n prod
```


## Clean Up

Let's uninstall the charts:

```
# List charts
helm list -n <namespace-name>

# Uninstall chart
helm uninstall <chart> -n <namespace-name>

# Example
helm uninstall mychart -n dev
```

Also, delete the namespaces we created:

```
# Delete namespace dev
kubectl delete ns dev

# Delete namespace prod
kubectl delete ns prod
```
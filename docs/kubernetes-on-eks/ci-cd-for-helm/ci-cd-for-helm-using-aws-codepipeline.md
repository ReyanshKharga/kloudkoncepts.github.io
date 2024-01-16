---
description: Learn how to implement Continuous Integration and Continuous Delivery (CI/CD) for Helm using AWS CodePipeline.
---

# CI CD for Helm Using AWS CodePipeline

Let's update our git repository to implement CI/CD for helm.


## Initialize the Chart

First, let's prepare helm template for our application:

```
helm create helm-chart
```

This will create a folder called `helm-chart` in the repository.

Remove everything from `templates/` folder and delete everything except `Chart.yaml`.

In the `Chart.yaml` change the `name` field to `node-app` or anything you prefer.

=== ":octicons-file-code-16: `Chart.yaml`"

    ```yaml linenums="1"
    apiVersion: v2
    name: node-app
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

Add `deployment.yaml` and `service.yml` templates in the `template/` directory.

=== ":octicons-file-code-16: `deployment.yaml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Values.appName }}
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
              - containerPort: {{ .Values.containerPort }}
    ```

=== ":octicons-file-code-16: `service.yaml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Values.appName }}
    spec:
      type: {{ .Values.service.type }}
      selector:
        app: {{ .Values.appName }}
      ports:
        - port: {{ .Values.service.port }}
          targetPort: {{ .Values.service.targetPort }}
    ```

Now, add `values.yam`l, `values.dev.yaml`, and `values.prod.yaml`.

=== ":octicons-file-code-16: `values.yaml`"

    ```yaml linenums="1"
    appName: myapp
    replicas: 1

    image:
      repository: IMAGE_REPOSITORY
      tag: IMAGE_TAG
      imagePullPolicy: Always

    service:
      type: ClusterIP
      port: 80
      targetPort: 5000

    containerPort: 5000
    ```

=== ":octicons-file-code-16: `values.dev.yaml`"

    ```yaml linenums="1"
    appName: myapp
    replicas: 1

    image:
      repository: IMAGE_REPOSITORY
      tag: IMAGE_TAG
      imagePullPolicy: Always

    service:
      type: ClusterIP
      port: 80
      targetPort: 5000

    containerPort: 5000
    ```

=== ":octicons-file-code-16: `values.prod.yaml`"

    ```yaml linenums="1"
    appName: myapp
    replicas: 2

    image:
      repository: IMAGE_REPOSITORY
      tag: IMAGE_TAG
      imagePullPolicy: Always

    service:
      type: ClusterIP
      port: 80
      targetPort: 5000

    containerPort: 5000
    ```

## Update Buildspec

Update the `buidspec.yml` to deploy using helm.

=== ":octicons-file-code-16: `buidspec.yml`"

    ```yaml linenums="1"
    version: 0.2
    run-as: root

    phases:

      install:
        commands:
          # Install helm
          - echo Installing helm...
          - curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
          - echo Verifying helm installation...
          - helm version --short
    
      pre_build:
        commands:
          # Verify if kubectl is installed
          - echo Checking if kubectl is installed...
          - kubectl version --client
          # Verify if aws-cli is installed
          - echo Checking if aws-cli is installed...
          - aws --version
          # Login to ECR
          - echo Logging in to Amazon ECR...
          - $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
          # GitHub commit hash to tag the image
          - COMMIT_HASH=$CODEBUILD_RESOLVED_SOURCE_VERSION
          # Additional latest tag
          - IMAGE_TAG='latest'

      build:
        commands:
          # Build the Docker image and add an additional latest tag
          - echo Building the Docker image...     
          - docker build -t $REPOSITORY_URI:$COMMIT_HASH .
          - docker tag $REPOSITORY_URI:$COMMIT_HASH $REPOSITORY_URI:$IMAGE_TAG

      post_build:
        commands:
          # Push the Docker images to ECR
          - echo Pushing the Docker image with commit hash as tag...
          - docker push $REPOSITORY_URI:$COMMIT_HASH
          - echo Pushing the Docker image with the latest image tag...
          - docker push $REPOSITORY_URI:$IMAGE_TAG
          - echo Pushed images to ECR.
          # Update kube config. This requires the codebuild role to have eks:DescribeCluster permission
          - aws eks update-kubeconfig --name $CLUSTER_NAME
          # Replace the IMAGE_REPOSITORY placeholder with ECR Repository URI
          - sed -i "s|IMAGE_REPOSITORY|$REPOSITORY_URI|g" helm-chart/values.$ENV.yaml
          # Replace the IMAGE_TAG placeholder with commit hash
          - sed -i "s|IMAGE_TAG|$COMMIT_HASH|g" helm-chart/values.$ENV.yaml
          # Create namespace if it doesn't exist
          - kubectl get ns $NAMESPACE || kubectl create ns $NAMESPACE
          # Upgrade chart. If a release by this name doesn't already exist, run an install.
          - echo Upgrading chart...
          - helm upgrade --install node-app helm-chart/ --values helm-chart/values.$ENV.yaml --namespace $NAMESPACE
    ```

In the Build Project, add `NAMESPACE` and `ENV` environment variables.

At this point your git repository folder structure should look like the following:

```
|-- nodeapp
│   |-- helm-chart
|   |   |-- templates
|   |   |   |-- deployment.yaml
|   |   |   |-- service.yaml
│   |-- k8s-manifests
|   |   |-- 00-namespace.yml
|   |   |-- deployment.yml
|   |   |-- service.yml
│   |-- .dockerignore
│   |-- .gitignore
│   |-- Dockerfile
│   |-- buildspec-k8s-manifest.yml
│   |-- buildspec.yml
│   |-- package-lock.json
│   |-- package.json
│   |-- server.js
```

Note that we have renamed the previous `buildspec.yml` to `buildspec-k8s-manifest.yml` and the updated `buildspec.yml` to work with helm.

Push changes to github and verify the deployment.


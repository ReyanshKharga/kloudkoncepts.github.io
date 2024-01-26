---
description: 
---

# Create Argo CD Application

An ArgoCD application is a Kubernetes Custom Resource Definition (CRD) used to define and manage the deployment of an application to a Kubernetes cluster.

An ArgoCD application resource defines the source repository, the target Kubernetes namespace, and the synchronization options, among other things.

You can create Argo CD application from the Argo CD UI or declaratively using the YAML file.

We'll set it up declaratively.


## Step 1: Prepare Kubernetes Manifest Files

Create a private git repo and add manifest files. I'll name the git repo kubernetes-manifests (You can name anything you want). In that repository I'll create a folder for node app called node-app.

Here's how the folder structure looks like:

```
|-- node-app
│   |-- 00-namespace.yml
│   |-- deployment.yml
│   |-- service.yml
```

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: nodeapp
    ```

=== ":octicons-file-code-16: `deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-nodeapp
      namespace: nodeapp
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nodeapp
      template:
        metadata:
          labels:
            app: nodeapp
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            ports:
              - containerPort: 5000
    ```


## Step 2: Create Argo CD Application

Now that we have our kubernetes manifest files ready, let's create Argo CD application.

=== ":octicons-file-code-16: `argo.master.yml`"

    ```yaml linenums="1"
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: node-app
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: https://github.com/ReyanshKharga/kubernetes-manifests.git
        targetRevision: master
        path: node-app
      destination:
        server: https://kubernetes.default.svc
        namespace: node-app
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
    ```

Apply the manifest to create the Argo CD application:

```
kubectl apply -f argo-node-app.yml
```

The application will be deployed but it will have an error since our repo is private. We need to configure argo cd to access git repo.

Credentials can be configured using Argo CD CLI:

```
argocd repo add https://github.com/argoproj/argocd-example-apps --username <username> --password <password>
```

Or you can use the UI. Navigate to `Settings/Repositories` and Click `Connect Repo using HTTPS` button and enter credentials.

Click `Connect` to test the connection and have the repository added.


You can also set up credentials to serve as templates for connecting repositories, without having to repeat credential configuration. For example, if you setup credential templates for the URL prefix `https://github.com/argoproj`, these credentials will be used for all repositories with this URL as prefix (e.g. `https://github.com/argoproj/argocd-example-apps`) that do not have their own credentials configured.


To set up a `credential template` using the Web UI, simply fill in all relevant credential information in the Connect repo using SSH or Connect repo using HTTPS dialogues (as described above), but select Save as credential template instead of Connect to save the credential template. Be sure to only enter the prefix URL (i.e. `https://github.com/argoproj`) instead of the complete repository URL (i.e. `https://github.com/argoproj/argocd-example-apps`) in the field Repository URL



## Argo CD Application for Helm

What if your kubernetes objects are defined as helm charts? How do you write Argo CD application for that?

Here's an example:

=== ":octicons-file-code-16: `argo.master.yml`"

    ```yaml linenums="1"
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: node-app-helm
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: https://github.com/ReyanshKharga/kubernetes-manifests.git
        targetRevision: master
        path: node-app-helm
        helm:
          valueFiles:
            - values.prod.yaml
      destination:
        server: https://kubernetes.default.svc
        namespace: node-app-helm
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
    ```

Here we have defined the repo, branch, path in the repo and which file to consider for helm values.


!!! quote "References:"
    !!! quote ""
        * [Connect Repo in Argo CD]{:target="_blank"}


<!-- Hyperlinks -->
[Connect Repo in Argo CD]: https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/
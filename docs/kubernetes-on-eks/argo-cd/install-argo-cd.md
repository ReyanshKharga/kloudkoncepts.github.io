---
description: Discover the basics of Argo CD, a powerful GitOps tool for managing Kubernetes applications. Learn how to create projects, applications, and configure authentication.
---

# Introduction to Argo CD

Let's see how you can install Argo CD in you kubernetes cluster.


## Step 1: Create Namespace for Arogo CD

First, let's create a namespace for Argo CD:

```
# Create argocd namespace
kubectl create namespace argocd
```


## Step 2: Install Argo CD

Now, let's install Argo CD in the `argocd` namespace that we created:

```
# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


## Step 3: Verify Argo CD Installation

```
# List pods
kubectl get pods -n argocd

# List services
kubectl get svc -n argocd
```


## Step 4: Access Argo CD on Local Machine

Let's use port forward to access Argo CD on local machine:

```
kubectl port-forward svc/argocd-server 8080:443 -n argocd
```

Visit `localhost:80` on your local host machine and you will be able to access Argo CD.



## Step 5: Retrieve Login Credentials

The initial password for the admin account/user is auto-generated and stored as clear text in the field `password` in a secret named `argocd-initial-admin-secret` in your Argo CD installation namespace.

Let's retrieve the password from the `argocd-initial-admin-secret` as follows:

```
kubectl get secrets argocd-initial-admin-secret -n argocd -o yaml
```

Copy the value of the `password` field from the output of the above command. The password is base64 encoded.

Let's decode the password:

```
echo <encoded-password> | base64 --decode
```

Now, use `admin` user with this password to login to Argo CD.



Argo CD also provides Argo CD CLI to perform operations using CLI. The CLI lets you interact with the Argo CD server using a terminal window.


## Step 6: Install Argo CD CLI

Use the following commands to install the Argo CD CLI:

```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

Verify CLI installation:

```
argocd version --client
```

Let's get the initial admin password using the Argo CD CLI:

```
argocd admin initial-password -n argocd
```

Login using CLI:

```
argocd login localhost:8080
```

List accounts and its details:

```
# List accounts
argocd account list

# Get user specific details
argocd account get --account admin
```


## Step 7: Create New User

Let's create new user `reyansh`. For this we need to update the `argocd-cm` ConfigMap.

```
# set editor to nano
export KUBE_EDITOR=nano

# Edit configmap
kubectl edit configmap argocd-cm -n argocd
```

Add the following in the `argocd-cm` ConfigMap:

```yaml
data:
  accounts.reyansh: apiKey, login
```

Verify if user was created:

```
# List accounts
argocd account list

# Get user specific details
argocd account get --account reyansh
```

Set user password:

```
argocd account update-password \
  --account <name> \
  --current-password <current-user-password> \
  --new-password <new-user-password>
```

If you are managing users as the admin user, `current-user-password` should be the current admin password.


As soon as additional users are created it is recommended to disable admin user. Update the configmap as follows:

```yaml
data:
  admin.enabled: "false"
```



!!! quote "References:"
    !!! quote ""
        * [Install Argo CD]{:target="_blank"}
        * [Install Argo CD CLI]{:target="_blank"}


<!-- Hyperlinks -->
[Install Argo CD]: https://argo-cd.readthedocs.io/en/stable/#getting-started
[Install Argo CD CLI]: https://argo-cd.readthedocs.io/en/stable/cli_installation/
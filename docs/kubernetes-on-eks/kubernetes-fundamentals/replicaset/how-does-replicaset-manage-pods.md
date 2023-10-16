# How Does ReplicaSet Manage Pods?

Standalone pods are like orphans. Nobody cares even if they die. Your application will be unavailable if the pod dies.

On the other hand, Pods managed by a `ReplicaSet` have a much better life. If for some reason they die, `ReplicaSet` will launch a new identical Pod. This ensures your application is available all the time.

But how does a `ReplicaSet` know which Pods to manage so that it can restart a Pod when required or kill the Pods that are not needed?

A `ReplicaSet` uses labels to match the pods that it will manage.

## Example

Consider the following `ReplicaSet`:

=== ":octicons-file-code-16: `my-replicaset.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: my-replicaset
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
            tier: backend
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 80
    ```


Here’s what happens when you apply the manifest to create the `ReplicaSet`:

1. The `ReplicaSet` controller checks for pods that match the labels defined in the `matchLabels` field of the `ReplicaSet` manifest. (`app=nginx` or `tier=backend`)
2. If such a Pod is found then the `ReplicaSet` controller checks if the Pod is already managed by another controller such as a `ReplicaSet` or a `Deployment`. (`ownerReferences` field in the Pod manifest can be used to find the owner of the object.)
3. If the Pod is not managed by any other controller, the `ReplicaSet` will start managing the Pod. Subsequently, the `ownerReferences` field of the target pods will be updated to reflect the new owner’s data (The `ReplicaSet` in this case).
4. The `ReplicaSet` will also launch new Pods if needed to maintain the stable set of replicas defined in the `ReplicaSet` manifest and update the `ownerReferences` field of those Pods.

Here's a visual representation of the flow described above:

<p align="left">
    <img src="../../../..//assets/eks-course-images/replicaset/how-does-replicaset-manage-pods.gif" alt="How Does ReplicaSet Mange Pods?" width="500" />
</p>

Let's see this in action.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}





<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx
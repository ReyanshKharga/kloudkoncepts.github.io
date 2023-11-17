---
description: Learn how to efficiently deploy microservices in Kubernetes. Our guide simplifies the process, offering step-by-step instructions for seamless implementation and management.
---

# Deploy Microservices in Kubernetes

Now that we have a good understanding of kubernetes and related AWS services, let's deploy a few microservices in our EKS kubernetes cluster.


## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/nodeapp:mongo]{:target="_blank"}
- [reyanshkharga/reactapp:v1]{:target="_blank"}
- [mongo:5.0.2]{:target="_blank"}

!!! note
    1. `reyanshkharga/nodeapp:mongo` is a Node.js backend application that uses MongoDB to store and retrieve data.

        Environment Variables:

        - `MONGODB_URI` (Required)
        - `POD_NAME` (Optional)
        
        Tha app has the following routes:

        - `GET /` Returns a JSON object containing `Host` and `Version`. If the `POD_NAME` environment variable is set, the value of the `Host` will be the value of the variable.
        - `GET /health` Returns health status of the app
        - `GET /random` Returns a randomly generated number between 1 and 10
        - `GET /books` Returns the list of books
        - `POST /addBook` Adds a book
        - `POST /updateBook` Updates the `copies_sold` value for the given book `id`.
        - `POST /deleteBook` Deletes the book record that matches the given id.

        Sample body for `POST /addBook`:

        ```json
        {
          "id": 1,
          "title": "Book1",
          "copies_sold": 0
        }
        ```

        Sample body for `POST /updateBook`:

        ```json
        {
          "id": 1,
          "copies_sold": 100
        }
        ```

        Sample body for `POST /deleteBook`:

        ```json
        {
          "id": 1
        }
        ```
    2. `reyanshkharga/reactapp:v1` is a React application. It's a frontend application that interacts with `reyanshkharga/nodeapp:mongo` backend to perform the CRUD operation.

        Environment variables:

        - `REACT_APP_API_ENDPOINT` (Optional)

        The environment variable `REACT_APP_API_ENDPOINT` is optional. If provided, you will be able to do the CRUD operations.

    3. `mongo:5.0.2` is MongoDB database. Our backend will use it to store and retrieve data to perform CRUD operations.



## Objective

We are going to deploy the following microservices on our EKS kubernetes cluster:

1. `MongoDB Database microservice`: uses docker image `mongo:5.0.2`
2. `Node.js Backend microservice`: uses docker image `reyanshkharga/nodeapp:mongo`
3. `React Frontend microservice`: uses docker image `reyanshkharga/reactapp:v1`


!!! note
    We will use the same load balancer for both backend and frontend microservices because using more load balancers will be expensive since load balancers are charged hourly. We can achieve this using [IngressGroup]{:target="_blank"}.



## Step 1: Deploy MongoDB Database Microservice

Let's create the kubernetes objects for our MongoDB database microservice as follows:

=== ":octicons-file-code-16: `00-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: mongodb
    ```

=== ":octicons-file-code-16: `deployment-and-service.yml`"

    ```yaml linenums="1"
    # Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mongodb-deployment
      namespace: mongodb
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mongodb
      template:
        metadata:
          labels:
            app: mongodb
        spec:
          containers:
          - name: mongodb
            image: mongo:5.0.2
            ports:
              - containerPort: 27017
            volumeMounts:
            - name: mongodb-storage
              mountPath: /data/db
          volumes:
          - name: mongodb-storage
            persistentVolumeClaim:
              claimName: mongodb-pvc
    ---
    # Service
    apiVersion: v1
    kind: Service
    metadata:
      name: mongodb-service
      namespace: mongodb
    spec:
      type: ClusterIP
      selector:
        app: mongodb
      ports:
        - port: 27017
          targetPort: 27017
    ```

=== ":octicons-file-code-16: `storageclass.yml`"

    ```yaml linenums="1"
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: mongodb-storageclass
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
      tagSpecification_1: "Name=eks-mongodb-storage"
      tagSpecification_2: "CreatedBy=aws-ebs-csi-driver"
    reclaimPolicy: Delete
    ```

=== ":octicons-file-code-16: `pvc.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mongodb-pvc
      namespace: mongodb
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: mongodb-storageclass
      resources:
        requests:
          storage: 4Gi
    ```

Assuming your folder structure looks like the one below:

```
|-- manifests
|   |-- mongodb
│   |   |-- 00-namespace.yml
│   |   |-- deployment-and-service.yml
│   |   |-- storageclass.yml
│   |   |-- pvc.yml
```

Let's apply the manifests to create the kubernetes objects for MongoDB database microservice:

```
kubectl apply -f mongodb/
```

This will create the following kubernetes objects:

1. A namespace named `mongodb`
2. A `StorageClass` (SC) for [dynamic provisioning]{:target="_blank"} of persistent volume
3. A `PersistentVolumeClaim` (PVC) in the `mongodb` namespace
4. MongoDB deployment in the `mongodb` namespace
5. MongoDB service in the `mongodb` namespace

!!! note
    The order in which yaml files are applied doesn't matter since every relation except namespace is handled by label selectors, so it fixes itself once all resources are deployed.

We are using Amazon EBS to persist the MongoDB data. EBS is provisioned dynamically using AWS EBS-CSI driver.

With [persistent volume] even if the MongoDB pod goes down the data will remain intact. When the new pod comes up we'll have the access to the same data.


Verify if the resources were created successfully:

```
# List all resources in mongodb namespace
kubectl get all -n mongodb

# List StorageClass
kubectl get sc

# List PersistentVolume
kubectl get pv

# List PersistenvVolumeClaim
kubectl get pvc -n mongodb
```

Verify if MongoDB is working as expected:

```
# Start a shell session inside the mongodb container
kubectl exec -it <mongodb-pod-name> -n mongodb -- bash

# Start the mongo Shell to interact with MongoDB
mongo

# List Databases
show dbs

# Switch to a Database
use <db-name>

# List collections
show collections
```




!!! quote "References:"
    !!! quote ""
        * [Kubernetes Resource Creation Order]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:mongo]: https://hub.docker.com/r/reyanshkharga/nodeapp
[reyanshkharga/reactapp:v1]: https://hub.docker.com/r/reyanshkharga/reactapp
[mongo:5.0.2]: https://hub.docker.com/_/mongo
[IngressGroup]: https://kloudkoncepts.com/kubernetes-on-eks/ingress/ingress-with-ingressgroup/
[dynamic provisioning]: https://kloudkoncepts.com/kubernetes-on-eks/kubernetes-fundamentals/storage-in-kubernetes/persistent-volume-using-amazon-ebs/dynamic-provisioning-of-pv-using-ebs/
[Kubernetes Resource Creation Order]: https://stackoverflow.com/a/56009748/10065458
[persistent volume]: https://kloudkoncepts.com/kubernetes-on-eks/kubernetes-fundamentals/storage-in-kubernetes/persistent-volumes/introduction-to-persistent-volumes/
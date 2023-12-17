---
description: Delete the book management microservices application we created and clean up the environment.
---

# Delete Book Management Microservices

Let's delete the resources we created for book management microservices and clean up our environment.


## Clean Up

Assuming your folder structure looks like the one below:


```
|-- manifests
|   |-- book-genres-db
│   |   |-- 00-namespace.yml
│   |   |-- configmap.yml
│   |   |-- deployment-and-service.yml
│   |   |-- storageclass.yml
│   |   |-- pvc.yml
|   |-- book-details-db
│   |   |-- 00-namespace.yml
│   |   |-- configmap.yml
│   |   |-- deployment-and-service.yml
│   |   |-- storageclass.yml
│   |   |-- pvc.yml
|   |-- book-genres
│   |   |-- 00-namespace.yml
│   |   |-- deployment-and-service.yml
│   |   |-- gateway.yml
│   |   |-- virtualservice.yml
|   |-- book-details
│   |   |-- 00-namespace.yml
│   |   |-- deployment-and-service.yml
│   |   |-- gateway.yml
│   |   |-- virtualservice.yml
|   |-- book-web
│   |   |-- 00-namespace.yml
│   |   |-- deployment-and-service.yml
│   |   |-- gateway.yml
│   |   |-- virtualservice.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/ --recursive
```


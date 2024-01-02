---
description: Explore the fundamentals of Amazon ECR with our comprehensive guide. Learn how to leverage Amazon Elastic Container Registry efficiently. Dive into ECR basics for seamless container image management.
---

# Introduction to Amazon ECR

Amazon Elastic Container Registry (Amazon ECR) is an AWS managed container image registry service that is secure, scalable, and reliable.


## Components of Amazon ECR

**Registry:** An Amazon ECR private registry is provided to each AWS account; you can create one or more repositories in your registry and store images in them.

**Authorization token:** Your client (Docker CLI in our case) must authenticate to Amazon ECR registries as an AWS user before it can push and pull images.

**Repository:** An Amazon ECR repository contains your Docker images.

**Image:** You can push and pull container images to your repositories.

**Repository policy:** You can control access to your repositories and the images within them with repository policies.


Now, let's see how you can create repositories in ECR and push images to it.



## Step 1: Create a Docker Image

First, create a Dockerfile as follows:

=== ":octicons-file-code-16: `Dockerfile`"

    ```Dockerfile linenums="1"
    FROM nginx:latest
    COPY ./index.html /usr/share/nginx/html/index.html
    ```

Next, build the image from the Dockerfile above:

```
docker build -t my-nginx-image .
```

Verify that the image was created correctly:

```
docker images --filter reference=my-nginx-image
```

Run a container from the image to verify the correctness of image:

```
docker run -it -p 3000:80 my-nginx-image
```

Open any browser and hit `localhost:3000` to view the html page from nginx Docker container.


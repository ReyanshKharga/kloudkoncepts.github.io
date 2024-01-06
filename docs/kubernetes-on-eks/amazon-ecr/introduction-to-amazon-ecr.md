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

=== ":octicons-file-code-16: `index.html`"

    ```html linenums="1"
    <!doctype html>
    <html>
        <head>
            <title>Nginx</title>
        </head>
        <body>
            <h2>Hello from Nginx container</h2>
        </body>
    </html>
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



## Step 2: Create Amazon ECR Repository

Create a repository to which we will later push the `my-nginx-image:latest` image.

```
# Command template
aws ecr create-repository \
    --repository-name <repository-name> \
    --image-scanning-configuration scanOnPush=true \
    --region <region-name>

# Actual command
aws ecr create-repository \
    --repository-name my-nginx-repository \
    --image-scanning-configuration scanOnPush=true
```


## Step 3: Authenticate to your ECR

Before we can push the images to Amazon ECR, we need to retrieve an authentication token and authenticate your Docker client to your registry. That way, the docker command can push and pull images with Amazon ECR.

The AWS CLI provides a `get-login-password` command to simplify the authentication process.

```
aws ecr get-login-password --region <region-name> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region-name>.amazonaws.com
```

Make sure to replace the `region-name` and `aws-account-id` placeholders with the appropriate values.


You can also get the `get-login-password` command from AWS Console by clicking View push commands.

Here's the output you will see if the command succeeds:

```
Login Succeeded
```


## Step 4: Push the Docker image to Amazon ECR

Prerequisites:

- The minimum version of docker is installed: 1.7
- The Amazon ECR authorization token has been configured with docker login.
- The Amazon ECR repository exists and the user has access to push to the repository.


List the images you have stored locally to identify the image to tag and push:

```
docker images
```

Tag the image to push to your repository:

```
docker tag my-nginx-image:latest <ecr-repository-uri>:<version>
```

Push the image to ECR:

```
docker push <ecr-repository-uri>:<version>
```


## Step 5: Pull an image from Amazon ECR

After your image has been pushed to your Amazon ECR repository, you can pull it from other locations like your local machine.

```
docker pull <ecr-repository-uri>:<version>
```


!!! quote "References:"
    !!! quote ""
        * [Amazon ECR]{:target="_blank"}
        * [Using Amazon ECR with the AWS CLI]{:target="_blank"}


<!-- Hyperlinks -->
[Amazon ECR]: https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html
[Using Amazon ECR with the AWS CLI]: https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html
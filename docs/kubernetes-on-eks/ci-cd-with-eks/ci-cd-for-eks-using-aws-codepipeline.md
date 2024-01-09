---
description: Learn how to streamline CI/CD for EKS with our comprehensive guide on leveraging AWS CodePipeline. Master automated deployment on Amazon EKS effortlessly.
---

# CI CD for EKS Using AWS CodePipeline

Now, let's see how you can set up CI/CD for a containerized application on EKS using AWS CodePipeline.


## Step 1: Create a Private GitHub Repository

In the first step let's create a private github repository. Let's call it `node-app`. You can name it anything you prefer.

Make sure to choose `gitignore` for Node.js application. This will create a `.gitignore` file for Node.js application.



## Step 2: Clone the GitHub Repository

Now, let's clone the repository on local machine so that we can make changes and push it to the repo.

```
git clone <github-repository-url>
```


## Step 3: Test the Application Locally

This is a simple Node.js app using express. Make sure you have `Node.js` and `npm` installed on your local machine so that you can test it locally.

```
# Verify if node is installed
node --version

# Verify if npm is installed
npm --version
```

=== ":octicons-file-code-16: `server.js`"

    ```js linenums="1"
    const express = require('express')
    const app = express()
    var os = require('os');

    const PORT = process.env.PORT || 5000;

    app.get('/', function (_req, res) {
      let host = os.hostname();
      data = {
        Host: host,
        Version: "v1"
      }
      res.status(200).json(data);
    });

    app.get('/health', function(_req, res) {
      res.status(200).send('Healthy');
    });

    app.listen(PORT, () => {
      console.log(`Server is running on port ${PORT}`);
    });
    ```

=== ":octicons-file-code-16: `package.json`"

    ```json linenums="1"
    {
      "name": "nodeapp",
      "version": "1.0.0",
      "description": "",
      "main": "server.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "dependencies": {
        "express": "^4.18.2"
      }
    }
    ```

Assuming your folder structure looks like the one below:

```
|-- nodeapp
│   |-- package.json
│   |-- server.js
```

Run the following command to install the npm packages:

```
npm install
```

Run the following command to test the app:

```
node server.js
```

Now, open another terminal and call localhost on port 5000.

```
# Call root endpoint
curl localhost:5000

## Call health endpoint
curl localhost:5000/health
```

Or, visit these URLs in any browser.



## Step 4: Containerize the App and Test it Locally

Let's create a `Dockerfile` and create a Docker image out of it. Also, add a `.dockerignore` file.


!!! note
    A `.dockerignore` is a configuration file that describes files and directories that you want to exclude when building a Docker image.


=== ":octicons-file-code-16: `Dockerfile`"

    ```Dockerfile linenums="1"
    FROM node:18

    # Create app directory
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json
    COPY package*.json ./

    # Install packages mentioned package.json
    RUN npm install

    # Bundle app source
    COPY . .

    EXPOSE 5000
    CMD [ "node", "server.js" ]
    ```

=== ":octicons-file-code-16: `.dockerignore`"

    ```yaml linenums="1"
    # Dependency directories
    node_modules/
    jspm_packages/

    # Environment variable files
    config.env
    .env
    .env.development.local
    .env.test.local
    .env.production.local
    .env.local

    # Logs
    logs
    *.log
    ```

Assuming your folder structure looks like the one below:

```
|-- nodeapp
│   |-- Dockerfile
│   |-- .dockerignore
│   |-- server.js
```

Run the following command to build the docker image:

```
docker build -t my-node-app .
```

Now, run a container from the docker image:

```
docker run --name my-node-container -p 5000:5000 my-node-app
```

Verify the application endpoints:

```
curl localhost:5000
curl localhost:5000/health
```

Stop and delete the container:

```
# list containers
docker ps -a | grep my-node-container

# stop the container
docker stop my-node-container

# Delete the container
docker rm my-node-container
```


## Step 5: Create Amazon ECR Repository

Create ECR repository to which we will later push the `my-node-app:latest` image that we built.

```
aws ecr create-repository \
    --repository-name my-node-app \
    --image-scanning-configuration scanOnPush=true
```


## Step 6: Create Kubernetes Manifest Files

Let's prepare kubernetes manifest files for our application.

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
            image: CONTAINER_IMAGE
            ports:
              - containerPort: 5000
    ```

=== ":octicons-file-code-16: `service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeapp-service
      namespace: nodeapp
    spec:
      selector:
        app: nodeapp
      ports:
        - port: 80
          targetPort: 5000
    ```

Note that `CONTAINER_IMAGE` is a placeholder that will be replace with actual ECR image URI during deployment.


## Step 7: Create Buildspec for AWS CodeBuild

A buildspec is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build.

=== ":octicons-file-code-16: `buildspec.yml`"

    ```yaml linenums="1"
    version: 0.2
    run-as: root

    phases:

      install:
        commands:
          - echo Nothing to Install.
          - echo AWS CodeBuild image already has the required tools we need.
          - echo We will verify it in the next stage.
    
      pre_build:
        commands:
          # Check if kubectl is installed
          - echo Checking if kubectl is installed...
          - kubectl version --client
          # Check if aws-cli is installed
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
          # Replace the CONTAINER_IMAGE placeholder with actual image URI
          - sed -i "s|CONTAINER_IMAGE|$REPOSITORY_URI:$COMMIT_HASH|g" k8s-manifests/deployment.yml
          # Apply any manifest changes in k8s-manifest folder. 
          - echo Applying kubernetes manifest changes...
          - kubectl apply -f k8s-manifests
    ```

The script performs the following actions:

1. Verifies whether the necessary dependencies, such as kubectl and aws-cli, are installed.
2. Logs into Amazon ECR.
3. Builds the Docker image.
4. Pushes the image to ECR.
5. Updates the deployment manifest by substituting the CONTAINER_IMAGE placeholder with the newly pushed image on ECR.
6. Deploys the kubernetes manifests to EKS.


At this point your folder structure should look like the following:

```
|-- nodeapp
│   |-- Dockerfile
│   |-- .dockerignore
│   |-- server.js
│   |-- package.json
│   |-- package-lock.json
│   |-- buildspec.yml
│   |-- k8s-manifests
        |-- 00-namespace.yml
        |-- deployment.yml
        |-- service.yml
```
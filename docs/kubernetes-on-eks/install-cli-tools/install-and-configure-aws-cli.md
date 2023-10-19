---
description: Learn how to install and configure AWS CLI for seamless cloud management. Step-by-step guide to getting started with Amazon Web Services.
---


# Install and Configure AWS CLI

The `AWS CLI` is a command-line tool that enables users to manage AWS services directly from the command line, offering a flexible and efficient way to create, configure, and interact with AWS resources.

Let's see how you can install the `AWS CLI` on your operating system and configure it to access resources in AWS.

!!! warning
    Given the ever-changing nature of the installation process, it is advisable to rely on the official documentation when installing `AWS CLI`.

    You can visit the [official documentation]{:target="_blank"} and follow the instructions to install or update the `AWS CLI` on your operating system.

## Step 1: Install AWS CLI

### Install AWS CLI on Mac

1. Download the installation file using the `curl` command:

    ```
    curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    ```

2. Run the standard macOS installer program, specifying the downloaded `.pkg` file as the source:

    ```
    sudo installer -pkg AWSCLIV2.pkg -target /
    ```

3. Verify the installation:

    ```
    # Verify that the shell can find and run the aws command in your $PATH
    which aws

    # Check AWS CLI version
    aws --version
    ```

### Install AWS CLI on Windows

1. Download and run AWS CLI MSI insatller

    To install AWS CLI on Windows you can just download and run the [AWS CLI MSI installer]{:target="_blank"} for Windows (64-bit).

3. Verify the installation:

    ```
    # Check AWS CLI version
    aws --version
    ```


### Install AWS CLI on Linux x86 (64-bit)

1. Download the installation file:

    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    ```

2. Unzip the installer:

    ```
    unzip awscliv2.zip
    ```

3. Run the installation program:

    ```
    sudo ./aws/install
    ```

4. Verify the installation:

    ```
    # Verify that the shell can find and run the aws command in your $PATH
    which aws

    # Check AWS CLI version
    aws --version
    ```


### Install AWS CLI on Linux ARM

1. Download the installation file:

    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
    ```

2. Unzip the installer:

    ```
    unzip awscliv2.zip
    ```

3. Run the installation program:

    ```
    sudo ./aws/install
    ```

4. Verify the installation:

    ```
    # Verify that the shell can find and run the aws command in your $PATH
    which aws

    # Check AWS CLI version
    aws --version
    ```


## Step 2: Configure AWS CLI

Just installing the `AWS CLI` won't grant you access to AWS resources. To interact with AWS services, you must configure it by providing your AWS credentials.

Below are the steps you can follow to configure the AWS CLI:

1. Create an AWS IAM user with programmatic access and download the access credentials (access key ID and secret access key).

2. Give the IAM user the necessary permissions. To keep it simple, we'll grant `Administrator` access to the user.

    !!! warning
        It's important to note that while we've simplified the process by granting `Administrator` access for convenience, it's generally recommended to provide more granular and specific permissions to IAM users based on their actual needs. This helps reduce potential security risks and ensures a more controlled and secure environment.

3. Configure the AWS CLI:

    ```
    # Command template
    aws configure --profile <aws-profile-name>

    # Actual command
    aws configure --profile my-aws-account
    ```

    You'll be prompted to enter `AWS Access Key ID` and `AWS Secret Access Key` and the Default `region` name this profile will use.

    !!! warning
        If you don't specify the `--profile` parameter, the `AWS CLI` will use the `default` profile. To enhance configuration and security, it's advisable to use a named profile when configuring the `AWS CLI`.

        Using a named profile also enables you to configure multiple AWS accounts with different settings.


## Step 3: Use AWS CLI to Access Resources in AWS

Now that we have configured the `AWS CLI`, let's explore how you can use it to access resources in your AWS account. 

For instance, you can list `S3` buckets using the following AWS CLI command:

```
aws s3 ls --profile <aws-profile-name>
```

You can also export the `AWS_PROFILE` environment variable and then call `aws` command without passing `--profile` parameter.

1. Export the `AWS_PROFILE` environment variable:

    ```
    export AWS_PROFILE=my-aws-account
    ```

2. Run AWS CLI command:

    ```
    aws s3 ls
    ```



!!! quote "References:"
    !!! quote ""
        * [Install or update the latest version of the AWS CLI]{:target="_blank"}
        * [Configure the AWS CLI]{:target="_blank"}
        * [Named Profile in AWS CLI]{:target="_blank"}


<!-- Hyperlinks -->
[official documentation]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
[Install or update the latest version of the AWS CLI]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
[AWS CLI MSI installer]: https://awscli.amazonaws.com/AWSCLIV2.msi
[Configure the AWS CLI]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html
[Named Profile in AWS CLI]: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
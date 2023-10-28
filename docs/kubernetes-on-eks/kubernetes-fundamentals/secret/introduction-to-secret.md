---
description: Get started with an easy introduction to Kubernetes Secrets. Learn how to securely manage sensitive information in your Kubernetes applications.
---

# Introduction to Kubernetes Secret

A `Secret` is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a pod specification or in a container image.

Using a `Secret` means that you don't need to include confidential data in your application code.

Because `Secrets` can be created independently of the pods that use them, there is less risk of the `Secret` (and its data) being exposed during the workflow of creating, viewing, and editing pods.

Secrets are similar to ConfigMaps but are specifically intended to hold confidential data. For example database credentials.

You can specify the `data` and/or the `stringData` field when creating a configuration file for a `Secret`.

The values for all keys in the `data` field have to be `base64-encoded` strings. If the conversion to `base64` string is not desirable, you can choose to specify the `stringData` field instead, which accepts arbitrary strings as values.


## Base64 Encoding and Decoding

Let's see how to `base64` encode and decode a string using bash:

```
# Encode text data
echo "reyansh" | base64

# Decode text data
echo "cmV5YW5zaAo=" | base64 --decode
```

!!! note
    A text can be encoded in two different `Base64` representations, but a single `Base64` encoding cannot have two distinct decodings.


## Use Cases of Secret

1. **Sensitive Data Storage:** `Secrets` securely store sensitive information like passwords, API keys, and certificates.

2. **Database Credentials:** `Secrets` manage and provide access to credentials required for connecting to databases securely.

3. **File Mounts:** `Secrets` can be used to mount confidential configuration files as volumes in pods, allowing applications to access sensitive data without including them in the container image.


!!! quote "References:"
    !!! quote ""
        * [Kubernetes Secrets]{:target="_blank"}


<!-- Hyperlinks -->
[Kubernetes Secrets]: https://kubernetes.io/docs/concepts/configuration/secret/
---
description: Explore YAML data types in our guide. Learn how YAML handles different data structures, making it a versatile choice for configuration and data serialization.
---

# Data Types in YAML

In YAML, data types refer to the different kinds of data that can be represented within a YAML document. YAML is a human-readable data serialization format often used for configuration files and data exchange between different programming languages. 

While YAML itself is not strongly typed like some programming languages, it provides a way to represent various data types, including strings, numbers, booleans, lists, maps (dictionaries), and more.


## Scalar Data Type

In YAML, a scalar is a single, atomic value. The value of the scalar can be integer, float, string, or boolean.

Here are a few examples of scalars:

```yaml
12
10.5
hello
hello world
"hello : world"
true
false
2023-12-10
```

!!! note
    You usually don't need quotes for YAML scalars, but if your data has special characters or could be misunderstood by YAML, use double ` or single ` quotes to keep it valid.

Here's a YAML document using scalars:

=== ":octicons-file-code-16: `student.yml`"

    ```yaml linenums="1"
    name: Reyansh Kharga
    age: 25
    weight: 67.8
    isPresent: true
    enrollmentDate: 2023-12-10
    ```


## Sequence Data Type

In YAML, a sequence is represented by a list of items.

There are two ways to represent a sequence in YAML:

1. Using a hyphen `-` followed by a space, and each item is placed on a new line with the same indentation level.
2. Using comma-separated items.

### Sequence Using Hyphen

=== ":octicons-file-code-16: `fruits.yml`"

    ```yaml linenums="1"
    fruits:
      - apple
      - banana
      - mango
    ```

### Sequence Using Comma-Separated Items

=== ":octicons-file-code-16: `fruits.yml`"

    ```yaml linenums="1"
    fruits: [apple, banana, mango]
    ```

### Nested Sequence

A nested sequence in YAML refers to a situation where you have a sequence that contains another sequence as one of its elements. 

In simpler terms, it's a list (or array) of items where one or more of those items is another list.

=== ":octicons-file-code-16: `fruits.yml`"

    ```yaml linenums="1"
    fruits:
      - apples
      - bananas
      - oranges
      - berries:
        - strawberries
        - blueberries
      - grapes
    ```


## Map Data Type

In YAML, the map data type is used to represent an unordered collection of key-value pairs. It is similar to a dictionary in other programming languages like Python.

Here are a few examples.

### 1. Simple Map

The following YAML code represents a map with four key-value pairs.

=== ":octicons-file-code-16: `student.yml`"

    ```yaml linenums="1"
    name: Reyansh
    age: 25
    weight: 67.8
    isPresent: true
    ```

### 2. Complex Map

Here's an example of a map with two key-value pairs. The key `friends` holds a list as its value.

=== ":octicons-file-code-16: `person.yml`"

    ```yaml linenums="1"
    name: Reyansh Kharga
    friends:
      - Adam
      - John
      - Sachin
    ```

### 3. Nested Map

In this example, `name` is a nested map with two key-value pairs.

=== ":octicons-file-code-16: `person.yml`"

    ```yaml linenums="1"
    name:
      firstName: Reyansh
      lastName: Kharga
    age: 25
    weight: 67.8
    ```

### 4. More Complex Map

In this example we have multiple level of nesting and includes values which are sequences.

=== ":octicons-file-code-16: `pod.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
      namespace: dev
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
        - name: busybox
          image: busybox:latest
    ```

Note that the item `containers` is a list of maps.

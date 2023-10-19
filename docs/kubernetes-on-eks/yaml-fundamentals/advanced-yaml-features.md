---
description: Discover advanced YAML features in our guide, including multiline strings, anchors, and aliases. Unlock powerful techniques for enhancing your YAML files.
---

# Advanced Features in YAML

Now that we're familiar with the basics of YAML, let's explore some advanced features like multiline strings, anchors and aliases, etc.


## Multiline String in YAML

In YAML, you can create a multiline string using the pipe character `|` or the greater than symbol `>`.


### Multiline String Using Greater Than Symbol

You can use `>` if you want the line breaks to be stripped out. You'll still have one line break at the end. You can use `>-` if you want to strip the line break at the end as well.

!!! tip
    You can use [YAML to JSON Converter Tool]{:target="_blank"} to see the resulting structure of the multiline string in YAML when it is parsed.

1. Multiline string using `>`:
    ```yaml
    # Using the greater than symbol
    multiline_string: >
      This is a 
      multiline 
      string 
      in YAML.
    ```

    YAML will treat the string as:

    ```yaml
    This is a multiline string in YAML.\n
    ```

2. Multiline string using `>-`:
    ```yaml
    # Using the greater than symbol
    multiline_string: >-
      This is a 
      multiline 
      string 
      in YAML.
    ```

    YAML will treat the string as:

    ```yaml
    This is a multiline string in YAML.
    ```

### Multiline String Using Pipe Character

You can use `|` if you want the line breaks to be preserved as `\n`. You can use `|-` if you want to strip the line break at the end.

1. Multiline string using `|`:
    ```yaml
    # Using the pipe character
    multiline_string: |
      This is a 
      multiline 
      string 
      in YAML.
    ```

    YAML will treat the string as:

    ```yaml
    This is a 
    multiline 
    string 
    in YAML.\n
    ```

2. Multiline string using `|-`:
    ```yaml
    # Using the pipe character
    multiline_string: |
      This is a 
      multiline 
      string 
      in YAML.
    ```

    YAML will treat the string as:

    ```yaml
    This is a 
    multiline 
    string 
    in YAML.
    ```


## Anchors and Aliases in YAML

YAML allows you to use `anchors` and `aliases` to create references to other parts of the document. This can be useful for avoiding repetition and simplifying complex structures.


### Anchor

An `anchor` is a name assigned to a specific value in the YAML document. It's indicated using the `&` character followed by the `anchor` name. For example:

```yaml
# Define an anchor for a list of values
list: &my_list
  - apple
  - banana
  - mango
```

In this example, the anchor name is `my_list` and it refers to the list of values `["apple", "banana", "mango"]`.

### Alias

An `alias` is a reference to an anchor in the YAML document. It's indicated using the `*` character followed by the `anchor` name. For example:

```yaml
# Use an alias to reference the list of values
fruits: *my_list
```

In this example, the `fruits` list contains an `alias` to the `my_list` anchor. When this YAML document is parsed, the `alias` will be replaced with the corresponding `anchor` value, resulting in the following list:

```yaml
fruits:
  - apple
  - banana
  - orange
```

### Using Anchor and Alias

Here's an example document that uses `anchor` and `alias`:

=== ":octicons-file-code-16: `my-document.yml`"

    ```yaml linenums="1"
    list: &my_list
      - apple
      - banana
      - mango

    fruits: *my_list
    ```

The above YAML is equivalent to the following YAML document:

=== ":octicons-file-code-16: `my-equivalent-document.yml`"

    ```yaml linenums="1"
    list:
      - apple
      - banana
      - mango

    fruits:
      - apple
      - banana
      - mango
    ```

!!! tip
    You can covert the first document to JSON and then convert the resulting JSON back to YAML to see if they are actually equivalent.


!!! quote "References:"
    !!! quote ""
        * [YAML to JSON Converter Tool]{:target="_blank"}
        * [JSON to YAML Converter Tool]{:target="_blank"}



<!-- Hyperlinks -->
[YAML to JSON Converter Tool]: https://onlineyamltools.com/convert-yaml-to-json
[JSON to YAML Converter Tool]: https://onlineyamltools.com/convert-json-to-yaml
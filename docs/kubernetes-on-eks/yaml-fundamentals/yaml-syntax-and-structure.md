---
description: Learn the syntax and structure of YAML with our comprehensive guide. Master the fundamentals of YAML for effective configuration and data representation.
---

# YAML Syntax and Structure

In YAML, indentation is used to indicate the structure of a document. 

!!! note
    Indentation refers to the number of spaces or tabs used to align elements in a YAML file.

In YAML, spaces are used for indentation, not tabs.

The file extension commonly used for YAML files is `.yaml` or sometimes `.yml`.

If you are using Visual Studio Code as your code editor, make sure to select `Indent using spaces`. This will ensure that the tabs are considered as spaces. (To indent using spaces, press ++cmd+shift+p++ on Mac or ++ctrl+shift+p++ on Windows and Linux, then search for `Indent using spaces`).


## Comment in YAML

In YAML, you can include comments by using the `#` symbol. Any text following the `#` symbol on a line will be treated as a comment and ignored by the parser.

=== ":octicons-file-code-16: `my-document.yml`"

    ```yaml linenums="1"
    # This is a comment
    key: value  # This is another comment
    ```

## Document Seperator in YAML

In YAML, `---` is used as a document separator. It is used to separate multiple documents within a single YAML file.

For example, if you have two YAML documents in a single file, you can use `---` to separate them, as shown below:

=== ":octicons-file-code-16: `my-document.yml`"

    ```yaml linenums="1"
    # YAML document 1
    key1: value1
    key2: value2
    ---

    # YAML document 2
    key3: value3
    key4: value4
    ```
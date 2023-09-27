# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

Deployment test : v1

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

```yaml title="values.yml"
name: Reyansh
age: 26
hello:
    world: xyz
    fruits:
    - apple
    - banana
```

!!! info "Options"

    * phone
    * email
    * profile
    * openid
    * aws.cognito.signin.user.admin

:smile:

:fontawesome-brands-twitter:{ .twitter }

:octicons-heart-fill-24:{ .heart }


!!! note "My note"

    This is a note


!!! Example "My Example"

    Hello this is my example
    ```yaml title="test.yml" linenums="1"
    name: Reyansh
    fruits:
        - apple
        - banana
    ```


!!! Example "My Example With Code Highlight"

    Hello this is my example
    ```yaml title="test.yml" linenums="1" hl_lines="2 3"
    name: Reyansh
    fruits:
        - apple
        - banana
    ```

??? Example "Collapsible example (Default is collapsed)"

    This example is collapsed by default


???+ Example "Collapsible example (Default is expanded)"

    This example is expanded by default


??? note "Collapsible note (Default is collapsed)"

    This note is collapsed by default


???+ note "Collapsible note (Default is expanded)"

    This note is expanded by default


[Subscribe to our newsletter](#){ .md-button }

[Subscribe to our newsletter](#){ .md-button .md-button--primary }

[Send :fontawesome-solid-paper-plane:](#){ .md-button }

```yaml
theme:
  features:
    - content.code.annotate # (1)
```

1. :man_raising_hand: I'm a code annotation! I can contain `code`, __formattedtext__, images, ... basically anything that can be written in Markdown.


``` py hl_lines="2 3" linenums="1"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

Done!

=== ":octicons-file-code-16: `deployment.yml`"

    ``` c linenums="1"
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== ":octicons-file-code-16: `service.yml`"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

=== ":octicons-file-code-16: `ingress.yml`"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

=== ":octicons-file-code-16: `gateway.yml`"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

=== ":octicons-file-code-16: `pv.yml`"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

=== ":octicons-file-code-16: `pvc.yml`"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```
..

=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

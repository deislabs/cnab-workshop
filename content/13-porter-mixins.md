# Porter Mixins

Mixins understand the Porter Manifest and the CNAB spec, and translate porter steps
into actions against another system such as Helm, Terraform or Azure.

This is how you get to avoid writing lots of bash. 😉

Initially we have 3 mixins:

* exec (this ships with porter)
* [helm](https://github.com/deislabs/porter-helm)
* [azure](https://github.com/deislabs/porter-azure)

## Exec Mixin
Execute commands and scripts. This helps you glue together steps, try stuff out,
or to help you reuse existing scripts.

```yaml
install:
- description: "Install Hello World"
  exec:
    command: bash
    arguments:
    - -c
    - echo Hello World
```

## Helm Mixin
Manage helm charts.

* Any secrets created by the chart are made available as outputs to the rest of the bundle.

## Azure Mixin
Work with Azure Cloud resources and services.

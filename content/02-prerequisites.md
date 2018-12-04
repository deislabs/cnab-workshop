# Prerequisites

Now that we're familiar with Cloud Native Application Bundles, it's time to set up our development environment and start using [Duffle][duffle], a reference implementation for CNAB:

- download [the latest version of Duffle for your platform][duffle-releases] and make sure the executable is available in your path
- install [Docker Community Edition for your platform][docker-ce]
- a Kubernetes cluster (see how to create an [Azure Kubernetes Service (AKS) cluster][create-aks])


> The `DOCKER_HOST` environment variable can be used to point to any Docker Engine instance, so any accessible Docker endpoint can be used with Duffle.

> [Instructions on how to activate an Azure Pass][azure-pass] in order to create an Azure Kubernetes Service.

To check if Duffle is installed locally, try to check the version. Depending on the version you downloaded, the output will be similar to:

```console
$ duffle version
0.1.0-ralpha.4+dramallamabuie
```

If you already have a Go development environment configured, you can also clone the repository and build the binary yourself. Instructions for this can be found in [the readme file for the Duffle GitHub repository][duffle-readme]

[duffle]: https://github.com/deislabs/duffle
[duffle-releases]: https://github.com/deislabs/duffle/releases
[docker-ce]: https://docs.docker.com/install/#supported-platforms
[azure-pass]: https://TODO
[create-aks]: https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough
[duffle-readme]: https://github.com/deislabs/duffle/blob/master/README.md

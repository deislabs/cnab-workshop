# Exercise: Create a bundle with Azure MySQL and Wordpress

This example bundle will create an Azure MySQL database instance and then use the credentials of that to install Wordpress. This bundle assumes you have an Azure account and a Kubernetes cluster created.  The bundle will use credentials for both.

First, install Porter!

## Install

### MacOS
```
curl https://deislabs.blob.core.windows.net/porter/latest/install-mac.sh | bash
```

### Linux
```
curl https://deislabs.blob.core.windows.net/porter/latest/install-linux.sh | bash
```

### Windows
```
iwr "https://deislabs.blob.core.windows.net/porter/latest/install-windows.ps1" -UseBasicParsing | iex
```

Next, clone the Porter repository to get the sample bundle.

```
git clone https://github.com/deislabs/porter.git
cd porter/examples/azure-mysql-wordpress
```

Open the porter.yaml file and take a look around. You'll also want to update the invocation image to reference a docker registry that you can push to. As part of the build step, you'll need to push to that registry.

For example, change

```
invocationImage: deislabs/porter-azure-wordpress:latest
```

to

```
invocationImage: <YOUR_DOCKERHUB>/porter-azure-wordpress:latest
```


Now, you can do a _porter build_

```
porter build
```

This will generate the Dockerfile and bundle.json for the example.

Now, you will need to setup your environment with the credentials for the bundle to use. To get the Azure credentials, first you'll need the Azure CLI.

## Install the Azure CLI

Install `az` by following the instructions for your operating system.
See the [full installation instructions](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) if yours isn't listed below. You will need az cli version 2.0.37 or greater.

### MacOS

```console
brew install azure-cli
```

### Windows

Download and run the [Azure CLI Installer (MSI)](https://aka.ms/InstallAzureCliWindows).

### Ubuntu 64-bit

1. Add the azure-cli repo to your sources:
    ```console
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
         sudo tee /etc/apt/sources.list.d/azure-cli.list
    ```
1. Run the following commands to install the Azure CLI and its dependencies:
    ```console
    sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
    sudo apt-get install apt-transport-https
    sudo apt-get update && sudo apt-get install azure-cli
    ```


Now, use the Azure CLI to obtain Azure credentials.

```
az login
```

```
az account list -o table
export AZURE_SUBSCRIPTION_ID=<sub-id>
```

```
az ad sp create-for-rbac --name porter-creds -o table

export AZURE_TENANT_ID=<TenantId>
export AZURE_CLIENT_ID=<AppId>
export AZURE_CLIENT_SECRET=<Password>
export AZURE_SUBSCRIPTION_ID=<sub-id>
```

Note, you'll also need to have the config for your kubernetes cluster.

Now, you can use duffle to generate a credential set for the bundle.

```
duffle credentials generate azure-mysql-wordpress -f bundle.json --insecure
```

This will prompt you to for a values for credentials in the bundle. Use the environment variables you set above.

Finally, you're ready to install the bundle:

```
duffle install porter-wordpress -c azure-mysql-wordpress -f bundle.json --insecure
```

# Using parameters and credentials

## Parameters

CNAB includes a method for declaring user-facing parameters that can be changed during certain operations (like installation). Parameters are specified in the build manifest (the bundle.json file). Duffle processes these as follows:

- The user may specify values when invoking duffle install or similar commands.
- Prior to executing the image, Duffle will read the manifest file, extract the parameter definitions, and then merge specified values and defaults into one set of finalized parameters
- During startup of the image, Duffle will inject each parameter as an environment variable, following the conversion method determined by CNAB:
- The variable name will be: CNAB_P_ plus the uppercase variable name - e.g. CNAB_P_SERVER_PORT (unless the destination field is set for the parameter), and the value will be a string representation of the value.

A CNAB bundle.json file MAY specify zero or more parameters whose values MAY be specified by a user.

As specified, values MAY be passed into the container as environment variables. If the environment variable name is specified in the destination, that name will be used:

```json
"parameters": {
    "greeting": {
        "defaultValue": "hello",
        "type": "string",
        "destination": {
            "env": "GREETING"
        },
        "metadata":{
            "description": "this will be in $GREETING"
        }
    }
}
```

The above will set `GREETING=hello`. In the case where no `destination` is set, a parameter is written as an environment variable with an automatically generated name - which can be accessed through the `$CNAB_P_<parameter-name>` environment variable.

Now let's take a look at a complete example that uses parameters. In our previous example, let's add the `parameters` section in the build manifest, `duffle.json`, so it becomes:

```json
{
    "name": "helloworld",
    "version": "0.1.0",
    "description": "A short description of your bundle",
    "keywords": [
        "helloworld",
        "cnab",
        "tutorial"
    ],
    "maintainers": [
        {
            "name": "John Doe",
            "email": "john.doe@example.com",
            "url": "https://example.com"
        },
        {
            "name": "Jane Doe",
            "email": "jane.doe@example.com",
            "url": "https://example.com"
        }
    ],
    "invocationImages": {
        "cnab": {
            "name": "cnab",
            "builder": "docker",
            "configuration": {
                "registry": "deislabs"
            }
        }
    },
    "parameters": {
        "greeting": {
            "defaultValue": "hello",
            "type": "string",
            "destination": {
                "env": "GREETING"
            },
            "metadata": {
                "description": "this will be in $GREETING"
            }
        }
    }
}
```

At this point, you have to simply use the `$GREETING` environment variable from your `run` executable - since in the simplest case it is a Bash script, let's just print it - so our `run` script becomes:

```bash
#!/bin/bash
action=$CNAB_ACTION
greeting=$GREETING

if [[ $action == "install" ]]; then
echo "hey I am installing things over here, $GREETING was passed as param"
elif [[ $action == "uninstall" ]]; then
echo "hey I am uninstalling things now, $GREETING was passed as param"
fi
```


Now if we rebuild the bundle and install it, we can pass the parameter using `--set greeting=<your-greeting>`:

```
$ duffle build
...
$ duffle install param-test helloworld --set greeting=HELLO
Executing install action...
hey I am installing things over here, HELLO was passed as param
$ duffle uninstall param-test helloworld --set greeting=GOODBYE
Executing uninstall action...
hey I am uninstalling things now, GOODBYE was passed as param
```


## Credential sets

Consider the case where a CNAB bundle named `example/myapp:1.0.0` connects to both ARM (Azure IaaS API service) and Kubernetes. Each has its own API surface which is secured by a separate set of credentials. ARM requires a periodically expiring token managed via `az`. Kubernetes stores credentialing information in a `KUBECONFIG` YAML file.

When the user runs `duffle install my_example example/myapp:1.0.0`, the operations in the invocation image need to be executed with a specific set of credentials from the user.

Those values must be injected from `duffle` into the invocation image - but Duffle must know which credentials to send.

The situation is complicated by the following factors:

- There is no predefined set of services (and thus predefined set of credentialing) specified by CNAB.
- It is possible, and even likely, that a user may have more than one set of credentials for a service or backend (e.g. credentials for two different Kubernetes clusters)
- Some credentials require commands to be executed on a host, such as unlocking a vault or regenerating a token
- There is no standard file format for storing credentials
- The consuming applications may require the credentials be submitted via different methods, including as environment variables, files, or STDIN.

Duffle solves this problem using a _credential set_ - named set of credentials (or credential generators) that is managed on the local host (via `duffle`) and injected into the invocation container on demand.

Now let's see how Duffle can help us generate credential sets, and how to use them.

First of all, clone the bundles repository from https://github.com/deislabs/bundles:

```
$ git clone https://github.com/deislabs/bundles
$ cd bundles/example-credentials
$ cat bundle.json

{
    "name": "example-credentials",
    "version": "0.0.1",
    "invocationImages": [
        {
        "imageType": "docker",
        "image": "cnab/example-credentials:latest"
      }
    ],
    "images": [],
    "credentials": {
        "secret1": {
            "env" :"SECRET_ONE"
        },
        "secret2": {
            "path": "/var/secret_two/data.txt"
        },
        "secret3": {
            "env": "SECRET_THREE",
            "path": "/var/secret_three/data.txt"
        }
    }
}
```

The bundle file describes what credentials it needs, and how they will be injected inside the invocation image: 

- `secret1` will be passed as the `$SECRET_ONE` environment variable
- `secret2` and `secret3` will be created as files, inside `/var/<secret-name>/data.txt`, as text files.

At this point, we can generate a credential set based on an existing bundle file:

> Note that because the bundle is unsigned, we have to pass the `--insecure` flag to `duffle`.


Duffle will prompt for the source of the credential set, and you can choose a specific value, the value of an environment variable, a file path or a shell command that generates the credential set.

```
$ duffle creds generate creds -f bundle.json --insecure
? Choose a source for "secret1" specific value
? Enter a value for "secret1" 46
? Choose a source for "secret2" environment variable
? Enter a value for "secret2" PATH
? Choose a source for "secret3" file path
? Enter a value for "secret3" ~/turtles.txt

$ duffle creds list
NAME    PATH
creds   /home/radu/.duffle/credentials/creds.yaml

$ duffle creds show creds
name: creds
credentials:
- name: secret1
  source:
    value: REDACTED
- name: secret2
  source:
    env: PATH
- name: secret3
  source:
    path: ~/turtles.txt # note that this file has to be present on your file system
```

Now we can use the credential set created when executing an action using this bundle:

> Note because a known bug in `duffle`, you may have to first pull the container invocation image using `docker pull cnab/example-credentials:latest`.

```
$ duffle install cred-test2 -f bundle.json -c creds --insecure
Executing install action...
SECRET_ONE: 46
/var/secret_two/data.txt
/home/radu/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin [REDACTED - rest of the expanded $PATH environment variable]
SECRET_THREE: turtles

/var/secret_three/data.txt
turtles
```

As this bundle simply echoes the values of the passed credential set, we can see how the credentials we passed were injected inside the invocation image.

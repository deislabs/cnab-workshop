# Building Cloud Native Application Bundles with Duffle

> Duffle is the first reference implementation for CNAB - because it follows the specification, any tool that implements the CNAB specification and outputs a valid bundle can be used to build bundles.

# Creating, building and installing bundles

Now that we have an understanding of what is a bundle, and our development environment is correctly setup, it's time to build our first bundle:

- first of all, we create a directory structure for our bundle:

```console
$ duffle create helloworld
==> Generating a new secret keyring at /home/radu/.duffle/secret.ring
==> Generating a new signing key with ID  <radu@computer-name>
==> Generating a new public keyring at /home/radu/.duffle/public.ring
Creating helloworld
```

This command initializes Duffle for the first time, then creates a new directory called `helloworld` (the only argument passed to the `duffle create` command). This directory contains the necessary directory structure to create our bundle:

- a build manifest file - `duffle.json` - this contains all required information to build our bundle
- a `cnab` directory, which contains a Dockerfile (used to build the invocation image) and an `app` directory, which in this case only contains a

```
$ cd helloworld && ls
cnab  duffle.json

$ ls cnab/
Dockerfile  app
```

> The specification describes [the bundle runtime][bundle-runtime] in detail - but in short, when executing an _action_ on a bundle (such as _install_, or _upgrade_), any tool that implements the CNAB specification must run the executable found in `/cnab/app/run` inside the invocation image. This needs the _action_ to be executed, and uses environment variables and files injected into the invocation image at runtime to execute the required action.


In this case, we have the simplest executable that satisfies the requirements for the bundle runtime - a Bash script - but keep in mind that any executable that correctly satisfies the requirements for the bundle runtime can be used as the run tool.

> Note that in a normal workflow for developing bundles, the author is not always supposed to write the run tool from scratch - there is already [an existing collection of bundles][bundles] for tools such as Terraform, Helm or Ansible, and when developing a bundle that uses these tools, these _base bundles_ can be used.

```bash
$ cat cnab/app/run

#!/bin/bash
action=$CNAB_ACTION
if [[ $action == "install" ]]; then
echo "hey I am installing things over here"
elif [[ $action == "uninstall" ]]; then
echo "hey I am uninstalling things now"
fi
```

At this point, we can execute the `duffle build` command, which will build the invocation image, together with the bundle (with information from the build manifest -`duffle.json`):

```
$ duffle build
Step 1/5 : FROM alpine:latest
 ---> 196d12cf6ab1
Step 2/5 : RUN apk add -u bash
 ---> Using cache
 ---> 8da8b360c44d
Step 3/5 : COPY Dockerfile /cnab/Dockerfile
 ---> Using cache
 ---> bcc0092953b7
Step 4/5 : COPY app /cnab/app
 ---> Using cache
 ---> fe379df2e706
Step 5/5 : CMD ["/cnab/app/run"]
 ---> Using cache
 ---> 0563a8f09431
Successfully built 0563a8f09431
Successfully tagged deislabs/helloworld-cnab:37db1697dab0116c78b3cc862fa105ebb380094d
==> Successfully built bundle helloworld:0.1.0

$ duffle bundle list
NAME            VERSION DIGEST                                          SIGNED?
helloworld      0.1.0   a35f1404b19f26890a4fc4f625cdef5d2cd953b5        true
```

We can now inspect the bundle manifest using the `duffle show` command:

```
$ duffle show helloworld
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
    "invocationImages": [
        {
            "imageType": "docker",
            "image": "deislabs/helloworld-cnab:37db1697dab0116c78b3cc862fa105ebb380094d"
        }
    ],
    "images": null,
    "parameters": null,
    "credentials": null
}
```

Now that we've built our very first bundle, it's time to install it - keep in mind that our very first bundle doesn't have any referenced images, parameters, or credentials:

```
$ duffle install our-first-test helloworld
Executing install action...
hey I am installing things over here

$ duffle status our-first-test
Installation Name:      our-first-test
Installed at:           2018-12-07 17:50:33.6694255 +0200 STD
Last Modified at:       2018-12-07 17:50:35.4166926 +0200 STD
Current Revision:       01CY4NT2MR7SC19AHC2ZQ7W517
Bundle:                 helloworld
Last Action Performed:  install
Last Action Status:     success
Last Action Message:
Executing status action in bundle...
```

And that's it - our bundle was executed using the run script we defined earlier, and it used the _install action_ defined by the Duffle CLI, in `duffle install`.


# Signed bundles

By default, all bundles built by Duffle are cryptographically signed, and by default, Duffle generates a signing key for you:

```
$ duffle bundle verify helloworld
Signed by "[anonymous key]" (D563 47CA C1BE BEF3 EBC4 1B75 B43F 79B2 BA39 F560)
```

You can add your own GPG secret keys to sign bundles, and share your public key so that others can verify bundles you built and signed:

```
$ duffle bundle verify <another-bundle>
Signed by "Your Name <user@email-provider.com>" (<signing-key-id>)
```

You can see the raw signed bundle file by executing:

```
$ duffle show helloworld --raw
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

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
  "invocationImages": [
    {
      "imageType": "docker",
      "image": "deislabs/helloworld-cnab:37db1697dab0116c78b3cc862fa105ebb380094d"
    }
  ],
  "images": null,
  "parameters": null,
  "credentials": null
}
-----BEGIN PGP SIGNATURE-----

wsFcBAEBCAAQBQJcBpr9CRD6XanxV8EQggAANSoQAGjgV4NfVJwKvAfBancDGFXF
AGL3kUEB/0eubPN5Ts0QXf1+k3OICdh06g+QTSrb6bM+ULwgxfUkZIDJjKGNBal4
04+/DW7MaX9WMW8L9SxKPJGwScNfk06j367kXhC0bRuKZajuHUseLJhzYGTRtwvn
vAS5x5PLYzHJBosDvO9Un9MZkrTH2N1I+zNXnS+CNLauG5yAiHWSmBnxxTxU1mvS
Ms2z6kGsxvF1GUrtjSnnPQ/G4g40DKQSs3NBZNO9gS0zTcaMKq5s3ZIBo1ivqu7X
KSSUFOeSjDQZdNOL/ylXQgNIYBxk7LzeVr1yz8zux9dDEkYhwjHBSYf5NtXYbe1E
JjvJku9SAqTZfsJvzlGLbrB5SUASvSJKS7coIVnRUTiV3UDb9kdrX9tUOEW9px13
2kasSu7q9pj2WnlptA3grQEmTvJsl9OdWH7ujHUVHTJQAN/pjMKfoGqFyLNFS96w
Tw6hS1bknYMts93vN4Ub7e6DFG27cOfTnx+5dxE7zwJ2YbD6TrATJ86cfT1HVU/u
JoVNPVHQLQ5u0taoS7lfL+1MsUEFrioTTUffxmww5OZ5mdPKt9TzP0UQ7X5I2R8a
9lJN7aJCyzvJcwz8TGFdOeFP1cjHqVwueniyxowdBWygkbbvTLPDDjQkjJN0Z7C+
nroJHDUoY0M0KxODijXB
=Nuqt
-----END PGP SIGNATURE-----
```

> This is the signed JSON file described in the specification.

[bundle-runtime]: https://github.com/deislabs/cnab-spec/blob/master/103-bundle-runtime.md
[bundles]: https://github.com/deislabs/bundles
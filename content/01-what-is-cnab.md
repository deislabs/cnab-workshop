# What is [CNAB]cnab?

> A more detailed description for Cloud Native Application Bundles can be found in [the official CNAB specification, on GitHub][cnab-spec].

[Cloud Native Application Bundles][cnab] (CNAB) are a _standard packaging format_ for multi-component distributed applications. It allows packages to target different runtimes and architectures. It empowers application distributors to package applications for deployment on a wide variety of cloud platforms, cloud providers, and cloud services. It also provides the capabilities necessary for delivering multi-container applications in disconnected environments.

CNAB is not a platform-specific tool. While it uses containers for encapsulating installation logic, it remains un-opinionated about what cloud environment it runs in. CNAB developers can bundle applications targeting environments spanning IaaS (like OpenStack or Azure), container orchestrators (like Kubernetes or Nomad), container runtimes (like local Docker or ACI), and cloud platform services (like object storage or Database as a Service). CNAB can also be used for packaging other distributed applications, such as IoT or edge computing.

The CNAB format is a packaging format for a broad range of distributed applications. It specifies a pairing of a _bundle definition_ [(`bundle.json`)](https://github.com/deislabs/cnab-spec/blob/master/101-bundle-json.md) to define the app, and an _invocation image_ to install the app.
The invocation image contains a standardized filesystem layout where metadata and installation data is stored in predictable places. A run tool is the executable entry point into a CNAB bundle. Parameterization and credentialing allow injection of configuration data into the invocation image. The invocation image is described in detail in the invocation image definition.


The _bundle definition_ is a single file that contains the following information:

- Information about the bundle, such as name, bundle version, description, and keywords
- Information about locating and running the _invocation image_ (the installer program)
- A list of user-overridable parameters that this package recognizes
- The list of executable images that this bundle will install
- A list of credential paths or environment variables that this bundle requires to execute

The bundle definition can:

- An unsigned JSON Object stored in a `bundle.json` file, as defined in [the bundle file definition](https://github.com/deislabs/cnab-spec/blob/master/101-bundle-json.md)
- A signed JSON object stored in a `bundle.cnab` file, as described in [the signature definition](https://github.com/deislabs/cnab-spec/blob/master/105-signing.md) - as a signed bundle definition represents an immutable bundle, all invocation images and images references must have a digest.



[cnab]: https://cnab.io
[cnab-spec]: https://github.com/deislabs/cnab-spec/
[bundle-json]: https://github.com/deislabs/cnab-spec/blob/master/101-bundle-json.md

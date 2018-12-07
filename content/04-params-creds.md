# Using parameters and credentials

CNAB includes a method for declaring user-facing parameters that can be changed during certain operations (like installation). Parameters are specified in the build manifest (the bundle.json file). Duffle processes these as follows:

- The user may specify values when invoking duffle install or similar commands.
- Prior to executing the image, Duffle will read the manifest file, extract the parameter definitions, and then merge specified values and defaults into one set of finalized parameters
- During startup of the image, Duffle will inject each parameter as an environment variable, following the conversion method determined by CNAB:
- The variable name will be: CNAB_P_ plus the uppercased variable name - e.g. CNAB_P_SERVER_PORT (unless the destination field is set for the parameter), and the value will be a string representation of the value.

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


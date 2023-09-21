# Intro

[Continue Here](https://getporter.org/quickstart/quickstart/)

[Porter](https://getporter.org/introduction/what-is-porter/) is an 
implementation of the Cloud Native Application Bundle specification. CNAB
is used to create bundles that can be used to install your application, 
together with required infrastructure and instance-specific configuration. 

A CNAB bundle is made up of three components:

- Application Images - typically docker images for your application
- Invocation Images - installer for the application
- Bundle Descriptor - metadata for the bundle: name, version, parameters, credentials, outputs

The CNAB bundle therefore allows you to package your application with everything needed to install it.
The bundle can be written to support air-gapped deployments: allowing installation 
without access to the original network used to develop the bundle and without access to the internet. 

The bundle can be an OCI-compliant artifact and therefore published to a OCI registry, such as 
Docker or Azure Container Registry

## Installation

Install porter for the current user:

```
export VERSION="latest"
curl -L https://cdn.porter.sh/$VERSION/install-linux.sh | bash
```

Then add `~/.porter` to your $PATH (e.g. by editing ~/.bashrc).

## Example Bundle

A porter bundle is defined in YAML. An example `porter.yaml` file 
will contain sections for each aspect of the bundle:

```yaml
schemaVersion: 1.0.0-alpha.1
name: examples/porter-hello
version: 0.2.0
description: "An example Porter configuration"
registry: ghcr.io/getporter

parameters:
  - name: name
    type: string
    default: porter
    path: /cnab/app/foo/name.txt
    source:
      output: name

outputs:
  - name: name
    path: /cnab/app/foo/name.txt

mixins:
  - exec

install:
  - exec:
      description: "Install Hello World"
      command: ./helpers.sh
      arguments:
        - install

upgrade:
  - exec:
      description: "World 2.0"
      command: ./helpers.sh
      arguments:
        - upgrade

uninstall:
  - exec:
      description: "Uninstall Hello World"
      command: ./helpers.sh
      arguments:
        - uninstall
```


## Parameters
Bundles can be authored to accept parameters, allowing the installation to 
be customised for each instance. Parameter values can be specified, in order
of priority, using:

1. Using the `--param` flag (or setting the parameters directly on an installation resource)
2. Using the `--parameter-set` flag (or setting the parameter set directly on an installation resource)
3. Re-using previous values from the last run of the bundle
4. Defaults specified in the bundle YAML definition

Use `porter explain` to review a bundles parameters. 
Explain will also tell which Actions the parameter can be used in (e.g. install, 
upgrade or uninstall). If the bundle has custom actions defined, explain will 
also indicate if the parameter can be used with those Actions also.

Parameters specified using the `--param` switch, expect a key-value pair:

`porter install --reference getporter/hello-llama:v0.1.1 --param name="Larry"`

For more complicated or sensitive bundles, parameter sets can be defined to 
provide the parameter values more securely and consistently. The 
`porter parameters generate` command can be used to create a 
parameter set:

`porter parameters generate hello-llama --reference getporter/hello-llama:v0.1.1`

The parameter set can be viewed with `porter parameters show`:

`porter parameters show hello-llama`

The parameter set can be specified with the `--parameter-set` switch:

`porter upgrade hello-llama --parameter-set hello-llama`

Because the `--param` flag takes precedence over `--parameter-set`, 
you can use to the `--param` flag to override one or more of the
values in a parameter set. 

Parameters values used during the last run of the bundle can be viewed
with:

`porter installation show`


## Credentials

Credentials required by a bundle can be specified in the the bundle
using the `credentials` key and referenced in the actions section
as `${bundle.credentials.CRED_NAME}`

Credential values can be sourced from environment variables, local files or from
an external secret store (if using the `secrets` plugin). 

Generate credentials with `porter credentials generate`:

`porter credentials generate github --reference getporter/credentials-tutorial:v0.1.0`

Use `list` or `show` to see the credentials defined on your workstation

The credential set can be specified with the `--credential-set` parameter:

`porter install --credential-set github --reference getporter/credentials-tutorial:v0.1.0`

Porter will re-use credentials from the previous run if no credential-set is 
specified. Unlike parameters, once the installation has run, credentials are
not stored in the installation history. 

**Credentials and Sensitive Parameters should be stored in key vault 
rather than environment variables or local files**


[Mixins](https://cdn.porter.sh/mixins/atom.xml) allow porter to work with additional tools such as Terraform, Azure, Kubernetes or Helm. The 'exec' mixin can be used to run custom scripts from a shell. Mixins are installed with:

`porter mixin install <mixin_name>`

You can begin to configure your Manifest using the `porter create` command. This will create a cnab directory and porter.yaml file in the current directory
Once the Manifest is configured the `porter build` command is used to build the bundle. Metadata is used to build the Bundle Descriptor. The Invocation image is built from the Steps defined in the Manifest. Runtimes are added based on 
specified mixins. Once the bundle is built it can be published to any OCI registry, 
e.g. dockerhub or ACR. 

[Plugins](https://cdn.porter.sh/plugins/atom.xml) are installed with:

`porter plugin install <mixin_name>`


View information about a bundle with `porter explain`:

`porter explain --reference getporter/porter-hello:v0.1.0`


To install the bundle use `porter install`:

`porter install porter-hello --reference getporter/porter-hello:v0.1.0`


List bundle installations with `porter list`. To see information
about an installation use `porter show`:

`porter show porter-hello`


You can upgrade an installation using `porter upgrade`:

`porter upgrade porter-hello --reference getporter/porter-hello:v0.1.1`


To clean up the resources installed from the bundle use `porter uninstall`:

`porter uninstall porter-hello`



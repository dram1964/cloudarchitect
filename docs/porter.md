# Intro
A CNAB bundle is made up of three components:

- Application Images - typically docker images for your application
- Invocation Images - installer for the application
- Bundle Descriptor - metadata for the bundle: name, version, parameters, credentials, outputs

The CNAB bundle therefore allows you to package your application with everything needed to install it.
Docker is used to package up the bundle

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


# Parameters
Bundles can be authored to accept parameters, allowing the installation to 
be customised. Use `porter explain` to review a bundles parameters. 
Explain will also tell which Actions the parameter can be used in (e.g. install, 
upgrade or uninstall). If the bundle has custom actions defined, explain will 
also indicate if the parameter can be used with those Actions also.
Parameters are specified using the `--param` switch:

`porter install --reference getporter/hello-llama:v0.1.1 --param name="Larry"`

For more complicated or sensitive bundles, parameter sets can be defined to 
provide the parameter values more securely and consistently. The 
`porter parameters generate` command can be used to create a 
parrameter set:

`porter parameters generate hello-llama --reference getporter/hello-llama:v0.1.1`

The parameter set can be viewed with `porter parameters show`:

`porter parameters show hello-llama`

The parameter set can be specified with the `parameter-set` switch:

`porter upgrade hello-llama --parameter-set hello-llama`


# Credentials
Credentials are used by individuals to identify themselves and execute 
bundles with their associated permissions. Unlike sensitive parameters, credentials
are not stored by porter and never reused by subsequent actions. Generate 
credentials with `porter credentials generate`:

`porter credentials generate github --reference getporter/credentials-tutorial:v0.1.0`

Use `list` or `show` to see the credentials defined on 
your workstation

The credential set can be used with the `--cred` parameter:

`porter install --cred github --reference getporter/credentials-tutorial:v0.1.0`

**Credentials and Sensitive Parameters should be stored in key vault 
rather than environment variables or local files**

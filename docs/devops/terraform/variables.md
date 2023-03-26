# Terraform Variables

Terraform variable blocks allow an existing configuration to be tailored 
using external input values. A variable block starts with the keyword 'variable' 
followed by the variable name and a block of attributes associated to the variable.
These attributes include:

- type ("string", "bool", "number", "object", "map", "list", "tuple", "sets", "any")
- default: a default value to use if none is supplied
- description: a text string explaining the use of the variable
- validation: a block with code used to validate the value supplied for the variable
- sensitive: a boolean value to determine if the variable value is displayed by the Terraform CLI

Variables and their default values can be declared anywhere in your configuration. 
However, it is preferable to create a separate 'variables.tf' so that others will know where to
look to customise the configuration.

Variables values can be assigned on the command-line:

`terraform plan -var azure_region="UK South"`

This becomes impractical if you want to override several default values. Instead you can 
additionally create a 'variable definitions' file. The variable definitions file should consist of only variable assignments:

```hcl
azure_region = "West Europe"
list_of_fruit = ["mango", "pineapple", "blueberry", "grape"]
res_tags = {
    location   = "UK South"
    department = "Accounts"
    owner      = "Freda Perry"
}
```

Terraform will automatically load variable defintion files named either 'terraform.tfvars' or 
files ending in '.auto.tfvars'. If you prefer to use JSON format for your variable definition files, 
Terraform will also load any files named 'terraform.tfvars.json' or ending in 'auto.tfvars.json'. 

```json
{
  "azure_region" : "West Europe",
  "list_of_fruit" : ["mango", "pineapple", "blueberry", "grape"],
  "res_tags" : {
    "location"   : "UK South",
    "department" : "Accounts",
    "owner"      : "Freda Perry"
  }
}
```

Variables can also assigned in environment variables by prefixing the variable name with "TF_VAR_":

```bash
export TF_VAR_location="UK South"
```

The corresponding variable "location" will need to be declared in your scripts to be accessible:

```hcl
variable "location" {
  default = ""
}

output "region" {
  value = var.location
}
```


## String Variables

Values for string variables are assigned with double-quoted strings:

```hcl
variable "azure_region" {
  type        = string
  default     = "UK West"
  description = "The Azure region to deploy resources"
}

output "az_region" {
  value = var.azure_region
}
```

Terraform supports string interpolation to allow the generation of strings by
combining variable values and string values. String interpolation expressions 
are declared within a double-quoted string and references to variable values 
or other resources are enclosed within '${}':

```hcl
name = "vm-${var.user_name}-${var.res_tags["department"]}"
```

Terraform also supports 'String Directives' which allow the generation of 
strings using loops or conditional statements. String directives are declared
within double-quoted strings and the control statements are enclosed within 
'%{}':

```hcl
output "users_created" {
  value = "%{ for val in var.user_names }${val} %{ endfor }"
}
```

Note that the string directive will output everything between the '%{}' blocks:
so that the space after '${val}' is also part of the output in the above example

String directives also support 'if' statements and HEREDOCS:

```hcl
enable_public_ip = <<EOT
%{ if var.location == "UK South" }
true
%{ else }
false
%{ endif }
EOT
```

## Bool variables

Boolean values can be stored in variables and used to toggle a feature in your 
configuration:

```hcl
variable "enable_ultra_ssd" {
    type    = bool
    default = false
    description = "Should UltraSSD_LRS disks be available"
}

resource "azurerm_virtual_machine" "my_vm" {
    ...
    ultra_ssd_enabled = var.enable_ultra_ssd
    ...
}
```

## List Variables

Lists are assigned as comma-separated lists inside square brackets:

```hcl
variable "private_subnet_cidr_blocks" {
    description = "Available cidr blocks for private subnets"
    type = list(string)
    default = [
        "172.30.1.0/24",
        "172.30.2.0/24",
        "172.30.3.0/24",
        "172.30.4.0/24",
        "172.30.5.0/24",
        "172.30.6.0/24",
    ]
}

variable "number_of_subnets_required" {
    description = "Number of subnets required"
    type    = number
    default = 2
}
```

Values can be retrieved from list variables by index or using the slice() function:

```hcl

output "third_private_subnet_block" {
    value = var.private_subnet_cidr_blocks[2]
}

output "first_three_subnets" {
    value = slice(var.private_subnet_cidr_blocks, 0, 3)
}

output "selected_subnets" {
    value = slice(var.private_subnet_cidr_blocks, 0, var.number_of_subnets_required)
}
```

## Map Variables

Map variables are collection types, and contain key-value pairs. The values should adhere to the 
declared datatype. Each key should be unique, where the keys and values are of type string:

```hcl
variable "res_tags" {
  type       = map(string)
  default = {
    location   = "UK West"
    department = "Finance"
    owner      = "Fred Perry"
  }
}

output "this_department" {
  value = var.res_tags["department"]
}
```

## Object Variables

Object variables are collection types, with key-value pairs where the values for each key can 
be of different types:

```hcl
variable "user_account" {
  type = object({
    logon    = string
    email    = string
    roles    = list(string)
    projects = map(string)
    enabled  = bool
  })
  default = {
    logon = "fred001"
    email = "fred@example.com"
    roles = ["user", "report writer", "remote access", "acr pull"]
    projects = {
      primary = "solaris"
      second  = "aix"
      third   = "rhel"
    }
    enabled = true
  }
}

output "user_details" {
  value = var.user_account.logon
}

output "main_project" {
  value = var.user_account.projects.primary
}
```

## Set Variables

Sets are collections that can contain only unique values of the same datatype:

```hcl
# Although the value "two" is included twice in the default assignment
# the value "two" will only occur once in the actual set variable
variable "myset" {
  type    = set(string)
  default = ["one", "two", "three", "four", "two"]
}

output "set_members" {
  value = var.myset
}
```

Elements of a set are identified only by their value and don't have any index or key to select
with, so it is only possible to perform operations across all elements of the set

## Tuple Variables

Tuples are similar to lists but each value in the list can be of a different type:

```hcl
variable "user_info" {
  type    = tuple([string, bool, number])
  default = ["Finance", true, 1]
}

output "tuple_values" {
  value = var.user_info
}

output "user_enabled" {
  value = var.user_info[1]
}
```

## Variable Validation

Validation blocks can be added to variable declaration to enforce any required rules and display
error messages if rules are broken. Functions like `length()` and `regexall()` can be 
useful for string validation. 

```hcl

variable "res_tags" {
...

    validation {
        condition = length(var.res_tags.["project"]) < 16 && length(regexall("[^a-zA-Z0-9]", var.res_tags.["project"])) == 0
        error_message = "Project tags must be less than 16 characters and only contain numbers or letters"
    }
...
}
```

## Protecting Sensitive Values

Variables can be assigned a 'sensitive' attribute to flag whether the value should be displayed
in the output from Terraform commands. If you create an output that includes the value of a 
variable flagged as sensitive, then this will cause terraform commands to fail, unless you also
flag the output as sensitive.

However, the terraform state file is stored as plain text, and even sensitive values are stored
as plain text so that terraform can read them to see if they have been changed. Therefore you 
should ensure that your terraform state file is kept secure at all times. 

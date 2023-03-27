# Terraform Functions

[Terraform Functions]("https://www.terraform.io/language/functions")

You can use the Terraform console to test out the various built-in functions 
of the Terraform language

## Ternary Operator

Terraform provides the traditional ternary operator '?:' to implement
conditional logic within blocks:
```hcl
variable "user_machines" = {
  type = map(string)
  default = {
    "vm_harry" : "rg-env-dev",
    "vm_larry" : "rg-env-live",
    "vm_barry" : "rg-env-test"
  }
}

resource azurerm_virtual_machine "user_vm" {
  for_each = var.user_machines
  name = each.key
  resource_group_name = each.value
  vm_size = (each.key == "vm_larry" ? "Standard_DS1_v2" : "Standard_FS1_v2")
...
...
}
```

## Merge

The merge operator can be used to join two or more maps together:
```hcl
locals = {
  default_tags = {
    "cost center" : "Central 234",
    "created_by" : "terraform"
  }
}

resource "azurerm_virtual_machine" "virtual_machine" {
...
  tags = merge(local.default_tags, var.custom_tags)
...
}
```

If the same key is defined in more than one map, the value from the right-most 
map is retained

## Lookup

The lookup operator can be used to retrieve a single value from a map using its
key:

```hcl
lookup(map, key, default)
```

## File

The `file()` function returns the content of the specified file

```hcl

resource "aws_instance" "web" {
...
  user_data = file("init_user.sh")
...
}
```

## CSVDecode

The csvdecode() function can be used to convert the rows in a CSV file to
elements in a list of maps. Use the file function to specify the file path:

```hcl
locals {
    users = csvdecode(file("${path.module}/users.csv"))
}
```

## For

'For' expressions can be used to iterate over any collection type to output 
another collection:

```hcl
{ for name, location in var.vm_map: lower(name) => lower(location) }
[ for name, location in var.vm_map: "${name}-${location}" ]
```

The new collection can be used to initialise a for_each iterator:

```hcl
for_each = { for user in local.users :  user.first_name => user }
```

## Format

The `format()` function can be used to generate strings, using a 
format string and values:

```hcl
user_principal_name = format(
  "%s%s-%s@%s",
  substr(lower(each.value.first_name), 0 , 1),
  lower(each.value.last_name),
  random_pet.suffix.id,
  local.domain_name
)
```



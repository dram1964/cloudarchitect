# Terraform Meta-Arguments

## Count

The 'count' attribute can be added to any resource and the value of this 
attribute determines the number of resource instances to be deployed. The 
'count' attribute provides an 'index' property which can be use with 
variable interpolation to assign unique names to each resource created. The 
index begins at 0.

```hcl
resource "azurerm_virtual_machine" "user_vm" {
  count = 3
  name = "${var.vm_name}-0${count.index}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
```

Terraform stores the list of resources created as an array in the terraform
state and the splatting operator (an asterisk) can be used to refer to this array as a 
whole.

The 'count' meta-element can also be used with a list variable, allowing more 
tailored names to be given to the resources created:

```hcl
variable "vm_names" {
  type = list(string)
  default = ["vm_harry", "vm_larry", "vm_barry"]
}

resource "azurerm_virtual_machine" "user_vm" {
  count = length(var.vm_names)
  name = "${var.vm_names[count.index]}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
```

## For Each

The 'for_each' meta element can be used to iterate a map or set 
to provision multiple resources from a single 
resource block. 'for_each' provides a 'each.key' and an 'each.value' attribute
to further tailor the properties of the resources created:

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
...
...
}
```

The 'for_each' meta-element can also be used to iterate at the property level: instead of using for_each to define multiple instance of the same resource type, for_each can be used to define multiple instances of a property on a singe resource:

```hcl
resource "azurerm_app_service" "app_service" {
  name = var.app_service_name
  location = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name
  app_service_plan_id = azurerm_app_service_plan.app_service_plan.id

  dynamic "connection_string" {
    for_each = var.connection_strings
      content {
        name = connection_string.name
        type = connection_string.type
        value = connection_string.value
      }
  }

  https_only = true
}
```

Property iterations use the keyword 'dynamic' along with the name of the 
property being iterated

## Depends On

Terraform can identify resource dependancies automatically, and will create
resources with no dependancies in parallel, whilst resources with dependancies
are created sequentially. But where a resource depends on another resources 
behaviour rather than it's data, we have to explicitly declare the dependancy. A 
data dependancy is clear to see: where the value for an attribute is declared by
reference to the value of an attribute of another resource. For example:

```hcl
resource azurerm_virtual_machine "my_database_server" {
...
  resource_group_name = azurerm_resource_group.resource_group.name
...
}
```

Implicit dependancies must be declared as a list of references to other 
resources:

```hcl
resource_azurerm_virtual_machine "my_app_server" {
...
  depends_on = [
      azurerm_virtual_machine.db_server.id,
      azurerm_virtual_machine.controller.id
  ]
}
```

## Provider

Where multiple providers are being used in Terraform scripts, you can add
a provider attribute to a resource to tell it to use the non-default provider

## Lifecycle

The 'lifecycle' meta-argument can be used to change terraform's default 
behaviour when creating, updating or destroying resources


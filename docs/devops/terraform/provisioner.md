# Provisioners
Terraform allows 'provisioner' blocks to be added to a resources attributes 
to enable execution of custom scripts or to copy files to the resource. There
are three built-in provisioner types:

- local-exec - used to execute scripts from the local machine
- file - used to copy files to the deployed resource
- remote-exec - used to execute scripts on the remote resource

The 'local-exec' scripts has one mandatory attribute (command) and three
optional attributes (working_dir, interpreter, environment, and when). 
Terraform also provides a 'null_resource' that can be used to execute a 
local script without associating the script execution to a specific resource:
```
resource "azurerm_resource_group" "resource_group" {
  name = var.resource_group_name
  location = var.location

  provisioner "local-exec" {
    command = "echo ${azurerm_resource_group.resource_group.id}"
  }
}

resource "null_resource" "my_web_app" {
  provisioner "local-exec" {
    command = "catalyst.pl"
    working_dir = "./bin"
    interpreter = ["perl", "-wc"]
    environment = {
      DEBUG = 1
      PORT = 8080
    }
  }
}
```
The local-exec provisioner also provides a 'when' attribute. If the value is
set to 'destroy' the command is only executed when 'terraform destroy' is 
called.
The 'file' provisioner is used to copy folders and files to the target 
resource:
```
provisioner "file" {
  source = "./resources/*.xml"
  destination = "/app/resources/"
}
```
The 'remote-exec' provisioner is used to execute scripts and commands on 
remote resources
```
provisioner "remote-exec" {
  inline = [
    "apt-get update",
    "apt-get install -y postgres"
  ]
}
```

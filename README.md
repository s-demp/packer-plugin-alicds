# Packer Plugin TSS

This plugin allows Packer to use a Thycotic Secret Server as a datasource.

## Installation

### Using pre-built releases

#### Using the `packer init` command

Starting from version 1.7, Packer supports a new `packer init` command allowing
automatic installation of Packer plugins. Read the
[Packer documentation](https://www.packer.io/docs/commands/init) for more information.

To install this plugin, copy and paste this code into your Packer configuration .
Then, run [`packer init`](https://www.packer.io/docs/commands/init).

```hcl
packer {
  required_plugins {
    tss = {
      version = ">= 0.1.0"
      source  = "github.com/breed808/packer-plugin-tss"
    }
  }
}
```

#### Manual installation

You can find pre-built binary releases of the plugin [here](https://github.com/breed808/packer-plugin-tss/releases).
Once you have downloaded the latest archive corresponding to your target OS,
uncompress it to retrieve the plugin binary file corresponding to your platform.
To install the plugin, please follow the Packer documentation on
[installing a plugin](https://www.packer.io/docs/extending/plugins/#installing-plugins).


#### From Source

If you prefer to build the plugin from its source code, clone the GitHub
repository locally and run the command `go build` from the root
directory. Upon successful compilation, a `packer-plugin-tss` plugin
binary file can be found in the root directory.
To install the compiled plugin, please follow the official Packer documentation
on [installing a plugin](https://www.packer.io/docs/extending/plugins/#installing-plugins).

## Usage

Add `data` blocks to use secrets from the Thycotic Secret Server. Note that a user account must be present in TSS with read access to the secret.

```hcl
data "tss" "mock-data" {
  username = "testing" # TSS username
  password = "test123" # TSS password

  secret_id = "500" # ID of TSS secret to retrieve
}
```

If the TSS user account uses LDAP for authentication, a `domain` must be specified:

```hcl
data "tss" "mock-data" {
  username = "testing" # TSS username
  password = "test123" # TSS password
  domain = "example.com" # Domain of user. I.E. testing@example.com

  secret_id = "500" # ID of TSS secret to retrieve
}
```

The TSS datasource will return `username` and `password` fields that can be used for authentication by Packer.
For example, a credential could be acquired for use with vSphere authentication:

```hcl
source "null" "example" {
  communicator = "none"
}

data "tss" "mock-data" {
  username = "testing" # TSS username
  password = "test123" # TSS password

  secret_id = "500" # ID of TSS secret to retrieve
}

build {
  sources = [
    "source.null.example"
    ]

    post-processors {
      post-processor "artifice" {
          files = ["output-vmware-iso/packer-vmware-iso.vmx"]
      }
      
      post-processor "vsphere" {
          keep_input_artifact = true
          vm_name    = "packerparty"
          vm_network = "VM Network"
          cluster    = "123.45.678.1"
          datacenter = "PackerDatacenter"
          datastore  = "datastore1"
          host       = "123.45.678.9"
          password   = data.mock-data.password
          username   = data.mock-data.username
      }
    }
}
```

## Packer Compatibility
This template is compatible with Packer >= v1.7.0

# Metadata

OpenStack Compute uses metadata to inject custom configurations to instances
on boot. You can add custom scripts, install packages, and add SSH keys to the instances
using metadata. The metadata can be accessed by `metadata service` or `config drive`.

# Metadata Service

The metadata service is accessed by the instance to fetch the instance specific
information by REST API. The metadata service runs on http://169.254.169.254
and it proxies the commands it receives, to a compute instance. At the boot time,
`cloud-init` calls metadata service to retrieve instance specific metadata to run
upon launch.

Openstack Neutron service runs a proxy that forwards metadata requests from instances
to the compute metadata service.

Nova metadata discovery via the Neutron metadata proxy is quite unreliable, as it is implemented
in a complex way and it fails sometimes. Issues can occur anywhere along the data flow of metadata
service architecture. In that case, you get an instance, but have missing metadata due to connection
timeout or unreachable metadata service which can lead to uninstalled packages or login issues.
So we recommend, a simple and more reliable alternative, which is using a config drive.

Now, we will see how to use the configuration drive to provide custom configurations to nova
instance.

# Store metadata on a configuration drive

A configuration drive is a read only drive that is attached to an 
instance during boot. The instance can then mount the drive and
read files from it. Configuration drives are used as a data source for
[cloud-init](https://cloudinit.readthedocs.io/en/latest/) and also for passing data to
your instances to configure.

# Enable the configuration drive

To enable the configuration drive, you need to pass the parameter `--config-drive`
with `true` flag to the `openstack server create` command. You can also
specify the user data you want to inject into the server using `--user-data` parameter.

## Example shell script

A bash script is a common way to inject user data to an instance.
The following script will install `tree` package during instance
creation.

#### **`myscript.txt`**
```bash
#cloud-config
package_update: true
packages:
  - tree
```

Now, we will pass the `myscript.sh` script using `--user-data` parameter.

```bash
openstack server create \
--config-drive true \
--user-data ./myscript.txt \
--image ubuntu \
--flavor m1.small \
--key-name [KEYPAIR] \
--nic net-id=[NETWORK_ID] \ 
guest0

```

This command creates the config drive with volume label `config-2` which is
attached to the instance during the boot and adds the contents of the file `myscript.sh`
to the `user_data` file in `openstack/{version}` directory.


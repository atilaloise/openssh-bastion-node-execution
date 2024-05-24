# OpenSSH Bastion Host Node Execution Plugins

This plugin provides a node-executor and file-copier supporting ssh actions through a bastion host.
Use this plugin if you must access remote servers via a jump host.

## Plugin Configuration Properties

* Bastion SSH Key Storage Path: Identity to use for the bastion host connection.
* SSH Options: Extra options to pass to the ssh command invocation. 
  You can overwrite this attribute at node level, using ssh-bastion-ssh-config (node-executor) and scp-bastion-ssh-config (file-copier).
* ssh_config: Specify ProxyCommand and other flags. Consult the reference for [ssh_config(5)](https://linux.die.net/man/5/ssh_config) to learn about posible settings.
* Dry run? If set true, just print the command invocation that would be used but do not execute the command. This is useful to preview.

## Node Specific Key

If the node is configured with the `ssh-key-storage-path` attribute, the ssh connection will use that to connect to the remote node.

* ssh-key-storage-path: Set to location in Rundeck Keystore

## Configuration 

The plugin can be configured as a default node executor and file copier for a Project. Use the Simple Conguration tab to see the configuration properties. The page has a form with inputs to configure the connection to the bastion host.

You can also modify the project.properties or use the API/CLI to define the plugin configuration.
The Plugin List page will describe the key names to set.

#### Customize the ssh_config 

You can define multiple lines using a trailing backslash and an indent on the following line.

Here is an example that defines ssh_config file. 

    project.plugin.NodeExecutor.openssh-bastion-host.node-executor.ssh_config=Host * \
      StrictHostKeyChecking no
      Port 22
      ProxyCommand ssh user@bastionhost -W %h\:%p
      IdentityFile @plugin.config.identity_file@

Here ssh_options are set.

    project.plugin.NodeExecutor.openssh-bastion-host.node-executor.ssh_options="-q -oCiphers=arcfour -oClearAllForwardings=yes"

Using Dry run, you might see output similar to this:

	[dry-run] +------------------------------------------+
	[dry-run] | ssh_config                               |
	[dry-run] +------------------------------------------+
	[dry-run] | Host *
	[dry-run] |   StrictHostKeyChecking no
	[dry-run] |   Port 22
	[dry-run] |   ProxyCommand ssh user@bastion -W %h:%p
	[dry-run] |   IdentityFile /tmp/bastion.ssh-keyfile.prWLUyFU
	[dry-run] +------------------------------------------+
	[dry-run] ssh -q -oCiphers=arcfour -oClearAllForwardings=yes -F /tmp/ssh_config.zTr9j5KK -i /tmp/host1234.ssh-keyfile.4cjnI2qL alexh@Targa.local whoami
	Begin copy 18 bytes to node host1234: /etc/motd -> /tmp/motd
	[dry-run] +------------------------------------------+
	[dry-run] | ssh_config                               |
	[dry-run] +------------------------------------------+
	[dry-run] | Host *
	[dry-run] |   StrictHostKeyChecking no
	[dry-run] |   Port 22
	[dry-run] |   ProxyCommand ssh user@bastion -W %h:%p
	[dry-run] |   IdentityFile /tmp/bastion.ssh-keyfile.XXXXX.WAlpZLNb
	[dry-run] | 
	[dry-run] +------------------------------------------+
	[dry-run] scp -q -oCiphers=arcfour -oClearAllForwardings=yes -F /tmp/ssh_config.XXXX.cosJ7xQ2 -i /tmp/host1234.ssh-keyfile.XXXXX.BOqYAKRu /etc/motd alexh@Targa.local:/tmp/motd
	/tmp/motd
	Copied: /tmp/motd



# Docker Example

To star the docker example:

* Run `./start-docker-example.sh`
* Got to `http://localhost:8080`
* User/Password => admin/admin

The example has two networks:

* Network1: rundeck, bastion
* Network2: bastion, linux-1 (running on port 2223), linux-2 (running on default port)

The goal of this example is that Rundeck connects to the nodes linux-1 and linux-2 through the bastion container (Rundeck cannot see linux-X nodes).
Notices the node attribute for "linux-1" node, we set the port connection on the "SSH Options" at the node level.

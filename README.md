# coreos-iso-maker V2.6

Many thanks to Stephen Nimmo (https://github.com/snimmo-redhat) for a Pull Request that now adds support
for FULL X.Y.Z releases.  This feature should allow you to specify whatever version of the CoreOS image
you want (EXCEPT 4.2 due to some weird naming changes but that is about to go EOL anyway.) 

New feature!  coreos-iso-maker now supports 2 NIC installs.  More should be possible but you are on your own for 
adding that.  :-)  Just follow the examples. *N.B.* The _may_ be an issue with the hostname being the same for
both network interfaces.  If anyone notices an issue, please speak up!

# Problem definition
Some customers would like to use static IPs for their OCP nodes but don't have a
working DHCP server for various reasons.  This can be done using the ISO for CoreOS
and messing with the boot parameters.  However, this is involves lots of error prone
typing.  This project is designed to work around that and originally created individualized ISOs
for each node.  In v2.1, it can be used to  create a single ISO with a menu item for each node being
built.  Due to screen space limitations, it is NOT recommend that you use this to create
more than 7 nodes at a time (1 bootstrap, 3 master control planes, and 3 worker nodes).
It _might_ work, but you might have problems with the display on boot.  Caveat user.
If you would prefer to have multiple ISOs, there is a separate playbook for that.

# Variables to define
In the `group_vars/all.yml` file, define the following variables:

`gateway`  	- default router IP
`gateway2`  	- default router IP for 2nd NIC (OPTIONAL)

`netmask`  	- default netmask
`netmask2`  	- default netmask for 2nd NIC (OPTIONAL)

`interface` 	- NIC device name.  Defaults to `ens192` which is default for VMWare
`interface2` 	- NIC device name. for 2nd NIC (OPTIONAL) Defaults to `en224` which is default for VMWare

`dns`		- dns server.  This can be done as a list.  Don't add more than 3.

`webserver_url` - webserver that holds the Ignition file

`webserver_port` - webserver port for the webserver above

`install_drive` - drive to install RHCOS on

`ocp_version` 	- Full OCP version you are going for. 4.4.3

`iso_checksum`	- sha256 checksum of the ISO.  Currently correct as of 2020-01-23 and OCP 4.3

`iso_name`	- Name of the ISO to download.  Makes certain assumptions that should be verified

`rhcos_bios`	- Name of the BIOS image to boot from.  Make certain assumptions that should be verified

In `inventory.yml` you will need to define your hosts:

`bootstrap`	- bootstrap node and its `ipv4` address

`masters`	- You will need to define `3` master nodes and their `ipv4` address

`workers`	- However many worker nodes you want and their corresponding `ipv4` addresses.  Recommend no more than 3 at a go.

There is a second example file called `inventory-multinic.yml` for an example of how to setup dual NIC machines.

You will need to use create individual ignition files and load them to your webserver.
This project does NOT currently do that.

Once the inventory is created, you can run either of the following commands:

For single:
`ansible-playbook playbook-single.yml -K`

For multiple:
`ansible-playbook playbook-multi.yml -K`

The included `ansible.cfg` file assumes the name of the inventory file is `inventory.yml`  Change it
in there if you want to use a different one or include the `-i <inventory-filename>` option to change it
on the command line.

The ISO(s) will be created in the `/tmp` directory.  The `-K` is to request for the BECOME password which is
required to mount an ISO (assuming you don't have passwordless `sudo`).

# Acknowlegements
Special thanks to Shanna Chan for trailblazing these issues (detail 
here: https://shanna-chan.blog/2019/07/26/openshift4-vsphere-static-ip/) and inspiring its creation.

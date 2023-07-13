# LINSTOR Ansible Playbook

Build a LINSTOR® cluster using Ansible. If you're unfamiliar with LINSTOR,
please refer to the
[Introduction to LINSTOR section](https://linbit.com/drbd-user-guide/linstor-guide-1_0-en/#p-linstor-introduction)
of the LINSTOR user's guide on https://linbit.com to learn more.

System requirements:

  - An account at https://my.linbit.com (contact sales@linbit.com).
  - Deployment environment must have Ansible `2.7.0+` and `python-netaddr`.
  - All target systems must have passwordless SSH access.
  - All hostnames used in the inventory file are resolvable (or use IP addresses).
  - Target systems are RHEL 7/8/9  or Ubuntu 22.04 (or compatible variants).

# Usage

Add the target system information into the inventory file named `hosts.ini`.
For example:
```
[controller]
192.168.35.12

[satellite]
192.168.35.[10:11]

[linstor_cluster:children]
controller
satellite

[linstor_storage_pool]
192.168.35.[10:11]
```

You can add a `controller` node to the `satellite` node group which will
result in the node becoming a `Combined` node in the LINSTOR cluster. A
`Combined` node will function both as a `controller` and as a `satellite` node.
Add nodes to the `linstor_storage_pool` node group to contribute block storage
to the LINSTOR storage pool created by the playbook.

Also, before continuing, edit `group_vars/all.yaml` to configure the necessary
variables for the playbook. For example:
```
---
# Ansible variables
ansible_user: vagrant
ansible_ssh_private_key_file: ~/.vagrant.d/insecure_private_key
become: yes

# LINSTOR variables
drbd_backing_disk: /dev/sdb
drbd_replication_network: 192.168.222.0/24

# LINBIT portal variables
lb_user: "lbportaluser"
lb_pass: "lbportalpass"
lb_con_id: "1234"
lb_clu_id: "4321"
```

The `drbd_backing_disk` variable should be set to an unused block device that the
LINSTOR satellite nodes will use if the nodes are also a part of the
`storage-pool` node group. If you do not have an unused block device, do not add
the node to the `linstor_storage_pool` node group, and only a `file-thin`
storage-pool will be configured instead. The `drbd_replication_network` is the network,
in CIDR notation, that will be used by LINSTOR and DRBD. It is strongly recommended
that the `drbd_replication_network` be separate from the management network in
production systems to limit network traffic congestion, but it's not a hard requirement.

When ready, run the `site.yaml` playbook:

```sh
ansible-playbook site.yaml
```

If you don't want to put your LINBIT credentials into the `group_vars/all.yaml`, you
can run the playbook like this instead:

```sh
ansible-playbook -e lb_user="username" -e lb_pass="password" -e lb_con_id="1234" -e lb_clu_id="1234" site.yaml
```

Congratulations! You should now have successfully created a LINSTOR cluster using Ansible.
# Testing Installation

Shell into the controller node, and check that everything is setup:

```sh
linstor node list; linstor storage-pool list
```
Create and deploy a resource:

```sh
linstor resource-definition create test-res-0
linstor volume-definition create test-res-0 100MiB
linstor resource create \
  $(linstor sp list | head -n4 | tail -n1 | cut -d"|" -f3 | sed 's/ //g') \
  test-res-0 --storage-pool lvm-thin
linstor resource list
```
You should now have a DRBD device provisioned on a node in your cluster that you
can use as you would any other block device.

# Reference

For more information on LINSTOR - such as instructions for Kubernetes,
OpenStack, Docker, or other integration - refer to
[LINBIT's LINSTOR documentation](https://linbit.com/drbd-user-guide/linstor-guide-1_0-en/).


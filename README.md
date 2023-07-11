# LINSTOR Ansible Playbook

Build a LINSTOR cluster using Ansible.

System requirements:

  - An account at https://my.linbit.com (contact sales@linbit.com).
  - Deployment environment must have Ansible `2.7.0+` and `python-netaddr`.
  - All target systems must have passwordless SSH access.
  - All hostnames used in inventory file are resolvable (or use IP addresses).
  - Target systems are RHEL 7/8/9  or Ubuntu 22.04 (or compatible variants).

# Usage

Add the system information gathered above into a file called `hosts.ini`.
For example:
```
[controller]
192.168.35.12

[satellite]
192.168.35.[10:11]

[linstor-cluster:children]
controller
satellite

[linstor-storage-pool:children]
satellite
```
You can list a `controller` in the `satellites` group which will result in the
node becoming a `Combined` node in the LINSTOR cluster.

Before continuing, edit `group_vars/all.yaml` for special options.

When ready, run the `site.yaml` playbook:

```sh
ansible-playbook site.yaml
```

If you don't want to put LINBIT credentials into your `group_vars/all.yaml`, you
can run the playbook like this instead:

```sh
ansible-playbook -e lb_user="username" -e lb_pass="password" -e lb_con_id="1234" -e lb_clu_id="1234" site.yaml
```

# Testing Installation

Shell into the controller node, and see that everything is setup:

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


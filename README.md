# LINSTOR Ansible Playbook

Build a LINSTOR cluster using Ansible.

System requirements:

  - Deployment environment must have Ansible `2.4.0+`
  - All target systems must have passwordless SSH access
  - All hostnames are resolvable
  

# Usage

Add the system information gathered above into a file called `hosts.ini`. For example:
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
You can list a `controller` in the `satellites` group which will result in the node becoming a `Combined` node in the LINSTOR cluster.

If you're planning on testing with Kubernetes, controller should be a k8s master and satellites k8s kubelets.

Before continuing, edit `group_vars/all.yaml` for special options.

After going through the setup, run the `site.yaml` playbook:

```sh
ansible-playbook site.yaml
```

If you don't want to put LINBIT credentials into your `group_vars/all.yaml`, you can run the playbook like this instead:

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
  test-res-0 --storage-pool thin-lvm
linstor resource list
```
You should now have a DRBD device provisioned on a node in your cluster that you can use as you would any other block device.

# Reference

For more information on LINSTOR - such as instructions for Kubernetes, OpenStack, Docker, or ProxMox integration - refer to [LINBIT's LINSTOR documentation](https://docs.linbit.com/docs/users-guide-9.0/#p-linstor).

# TODO

  - Complete or remove Debian/Ubuntu support

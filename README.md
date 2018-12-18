# LINSTOR Ansible Playbook

Build a LINSTOR cluster using Ansible with LINSTOR.

System requirements:

  - Deployment environment must have Ansible `2.4.0+`
  - All target systems must have passwordless SSH access
  - All hostnames are resolveable
  

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
If you're planning on testing with Kubernetes, controller should be a k8s master and satellites k8s kubelets.

Before continuing, edit `group_vars/all.yaml` for special options.

After going through the setup, run the `site.yaml` playbook:

```sh
$ ansible-playbook site.yaml
```

If you don't want to put LINBIT credentials into your `group_vars/all.yaml`, you can run the playbook like this instead:

```sh
$ ansible-playbook -e lb_user="username" -e lb_pass="password" -e lb_con_id="1234" -e lb_clu_id="1234" site.yaml
```

# Testing Installation

Shell into the controller node, and see that everything is setup:

```sh
# linstor n l; linstor sp l 
```

If you're going to use LINSTOR with Kubernetes, you'll need to install the flexvolume plugin on all nodes:

```sh
# yum install linstor-external-provisioner linstor-flexvolume-kubernetes
  -- OR-
# apt install linstor-external-provisioner linstor-flexvolume-kubernetes
```

Then, shell into the controller and start the provisioner:

```sh
# screen -dmS linstor-provisioner bash -c "/usr/sbin/linstor-external-provisioner -provisioner=external/linstor -kubeconfig=/etc/kubernetes/admin.conf"
```

Then, define the k8s storage class for the linstor storage pool from the playbook. For example:

```sh
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: two-replica-autoplace-thin
provisioner: external/linstor
parameters:
  autoplace: "2"
  storagePool: "thin-lvm"
```

Finally, apply it to k8s:

```sh
$ kubectl create -f /root/two-replica-autoplace-sc.yaml
```

After that you can request persistent volumes for k8s! For example:

```sh
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-0-two-replica-autoplace-thin
  annotations:
    volume.beta.kubernetes.io/storage-class: two-replica-autoplace-thin
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

# TODO

  - Complete Debian/Ubuntu (just firewall rules?)
  - Add instructions/options for containerized LINSTOR controller

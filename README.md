# Build a Kubernetes cluster using *k3s* with *ansible*

Author: <https://github.com/itwars>

## Introduction to *k3s-ansible*

The goal of *k3s-ansible* is to easily install a Kubernetes cluster on a variety of
operating systems running on machines with different architectures.
In general, users of *k3s-ansible* should only need to edit two files:
- `inventory/sample/group_vars/all.yml`
- `inventory/sample/hosts.ini`

All you need to get started is a list of IP addresses for the hosts that you want to
participate in the cluster and a username that has password-less *ssh* access to all
those hosts.  That's it!
No need to futz with lots of settings and variables (unless you like that sort of thing;
then, have at it).

And, to setup an HA cluster, you need one more IP address - not of a host,
but for your cluster virtual IP address.
You don't need to know how to setup a clustering solution since *k3s-ansible* does it for you.
But, for HA, you just need at least three hosts.

The intention is for *k3s-ansible* to support what *k3s* supports.\
Here is what has been tested (:heavy_check_mark:) with *k3s-ansible*.

| Operating System | amd64 | arm64 | armhf |
| :--------------- | :---: | :---: | :---: |
| Debian           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Ubuntu           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| CentOS           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| RedHat           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| ArchLinux        | implemented | implemented | implemented |

## System requirements

- The deployment environment must have *ansible* v2.4.0+.
- Hosts in the cluster must have password-less *ssh* access.
- HA requires at least three hosts.

##  Caveats

- *k3s-ansible* will overwrite an existing *k3s* installation on the hosts.
- *k3s-ansible* will overwrite the `.kube` directory of the `ansible_user` specified on each server.
- An HA configuration using *Keepalived* will overwrite an existing *Keepalived* configuration.

## Usage

1. Create a new cluster definition based on the `inventory/sample` directory.

```bash
cp -R inventory/sample inventory/my-cluster
```

2. Edit `inventory/my-cluster/hosts.ini` to include the hosts that will make up your new cluster.\
For example:

```bash
[k3s_server]
192.16.35.12

[k3s_agent]
192.16.35.[10:11]

[k3s_cluster:children]
k3s_server
k3s_agent
```

3. Edit `inventory/my-cluster/group_vars/all.yml` to best match your environment.\
See, `inventory/sample/group_vars/README.md` for more details.

4. Provision your new cluster.

```bash
ansible-playbook playbook/site.yml -i inventory/my-cluster/hosts.ini
```

## Kubeconfig

To get access to your new **Kubernetes** cluster, just use the generated kube configuration file.

```bash
kubectl --kubeconfig playbook/cluster.conf ...
```

## High Availability
*k3s-ansible* can now configure a high-availability (HA) cluster.
If you enable HA (**ha_enabled**), the playbook will setup an embedded database using *etcd*.
HA requires at least version **v1.19.5+k3s1** and an odd number of servers (minimum of three).
See the [HA-embedded documentation](https://rancher.com/docs/k3s/latest/en/installation/ha-embedded/) for more details.

HA expects that there is a virtual IP (**ha_cluster_vip**) in front of the *control-plane* servers.
A few methods have been implemented to provide and manage this VIP.
See `inventory/turingpi` for my example HA setup on my Turing Pi v1.
See `inventory/sample/group_vars/README.md` for more details on variables.


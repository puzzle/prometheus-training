---
title: "Debian/Ubuntu Vagrant setup"
description: "Vagrant installation on Debian/Ubuntu Linux"
weight: 3
type: docs
sectionnumber: 1
---

## Debian/Ubuntu Vagrant setup

To participate in the lab you can use any Linux server
of your choice. The labs are tailored for setup with
CentOS/RHEL hosts. Below are instructions on how to setup
the required host with [Vagrant][vagrant] on Linux.
Follow the step-by-step guide to bootstrap the techlab
environment on your OS of choice.


### Connectivity details

{{% alert title="Note" color="primary" %}}
The following passwords are not secure and intended only to
be used with local virtual machines not reachable from outside
of the virtualization host.
{{% /alert %}}

Linux Vagrant setup provides a local
CentOS virtual machine running under KVM with the
following IP addresses and credentials.

```yaml
control: 192.168.122.60

user: vagrant
password: vagrant
```


### Debian/Ubuntu-based systems

{{% alert title="Note" color="primary" %}}
Debian and Ubuntu ship a Vagrant package by default.
Depending on the version and age of the distribution
Vagrant may not include support for CentOS 8 and fails
during the initial setup. As such, the Debian package
from HashiCorp (the vendor of Vagrant) is used to
ensure a frictionless lab experience.

During the package installation the current user is added to
the `libvirt` group. In case you get a `permission denied` error
during `vagrant up`, you must terminate your session and log in
again.

If you still have troubles you can use [this guide for troubleshooting](https://ubuntu.com/server/docs/virtualization-libvirt>).
{{% /alert %}}


#### Installation and startup

Install `libvirt` and dependencies:

```bash
sudo apt install libvirt-daemon libvirt-clients libvirt-dev qemu-kvm libvirt-daemon-system
```

Install Vagrant from HashiCorp:

```bash
curl --location -o /var/tmp/vagrant_2.2.14_x86_64.deb https://releases.hashicorp.com/vagrant/2.2.14/vagrant_2.2.14_x86_64.deb
sudo dpkg -i /var/tmp/vagrant_2.2.14_x86_64.deb
```

Install the `libvirt` Vagrant plugin:

```bash
vagrant plugin install vagrant-libvirt
```

Create the working directory and download the Vagrantfile:

```bash
mkdir prometheus-labs
cd prometheus-labs
curl -o Vagrantfile https://raw.githubusercontent.com/puzzle/prometheus-labs/main/Vagrantfile
```

Start the virtual machine:

```bash
vagrant up
```

Access it:

```bash
ssh vagrant@192.168.122.60
```

#### Shutdown

Switch to the working directory and stop the virtual machine:

```bash
cd prometheus-labs
vagrant destroy -f
```

[vagrant]: https://www.vagrantup.com/

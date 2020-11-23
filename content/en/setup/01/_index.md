---
title: "CentOS/RHEL Vagrant Setup"
description: "Vagrant installation on CentOS/RHEL Linux"
weight: 3
type: docs
sectionnumber: 1
---

## CentOS/RHEL Vagrant Setup

To participate in the lab you can use any Linux server
of your choice.  The labs are tailored for setup with
CentOS/RHEL hosts. Below are instructions on how to setup
the required host with [Vagrant][vagrant] on Linux.
Follow the step by step guide to bootstrap the techlab
environment on your OS of choice.

{{% alert title="Note" color="primary" %}}
The following passwords are not secure and intended only to
be used with local virtual machines not reachable from outside
of the virtualization host.
{{% /alert %}}

### Connectivity Details

Linux Vagrant setup provides a local
CentOS virtual machine running under KVM with the
following IP addresses and credentials.

```yaml
control: 192.168.122.60

user: vagrant
password: vagrant
```

### CentOS/RHEL Based Systems

{{% alert title="Note" color="primary" %}}
RHEL and CentOS ship per default with Vagrant.
Depending on the version and age of the distribution
Vagrant may not include support for CentOS 8 and fails
during the initial setup. As such the RPM package
from HashiCorp the vendor of Vagrant is utilized to
ensure a frictionless lab experience.
{{% /alert %}}

#### Techlab Installation and Startup

```bash
# install libvirt and build-dependencies
sudo yum install libvirt libvirt-daemon-kvm libvirt-devel gcc make rsync

# start libvirtd
sudo systemctl start libvirtd.service

# install vagrant from hashicorp
curl --location -o /var/tmp/vagrant_2.2.14_x86_64.rpm \
  https://releases.hashicorp.com/vagrant/2.2.14/vagrant_2.2.14_x86_64.rpm
sudo yum localinstall /var/tmp/vagrant_2.2.14_x86_64.rpm

# Add your user to the libvirt group
sudo usermod -a -G libvirt ${USER}

# install vagrant plugin for libvirt
vagrant plugin install vagrant-libvirt

# create working directory and download vagrant file
mkdir prometheus-labs
cd prometheus-labs
curl -o Vagrantfile \
  https://raw.githubusercontent.com/puzzle/prometheus-labs/main/Vagrantfile

# setup vm
vagrant up

# access vm
ssh vagrant@192.168.122.60
```

#### Techlab Shutdown

```bash
cd prometheus-labs

# shutdown all vm
vagrant destroy -f
```

[vagrant]: https://www.vagrantup.com/

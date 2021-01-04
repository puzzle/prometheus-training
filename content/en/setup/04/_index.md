---
title: "macOS Vagrant setup"
description: "Vagrant installation on macOS"
weight: 4
type: docs
sectionnumber: 1
---

## Vagrant setup on macOS

To participate in the lab you can use any Linux server of your choice. The labs are tailored for a setup with CentOS/RHEL hosts.
Below are instructions on how to set up the required host with Vagrant on macOS.
Follow the step-by-step guide to bootstrap the lab environment on your OS of choice.

### Connectivity details

{{% alert title="Note" color="primary" %}}
The following passwords are not secure and intended only to
be used with local virtual machines not reachable from outside
of the virtualization host.
{{% /alert %}}

The Vagrant setup provides a local CentOS VM with the following IP addresses and credentials.

```yaml
control: 192.168.122.60

user: vagrant
password: vagrant
```


### Installation and startup

Either install Vagrant with [Homebrew](https://formulae.brew.sh/cask/vagrant) or follow the installation instructions from the [Vagrant website](https://www.vagrantup.com/downloads).

Homebrew command:

```bash
brew install --cask vagrant
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

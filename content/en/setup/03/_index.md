---
title: "Windows Vagrant Setup"
description: "Vagrant installation on Windows"
weight: 3
type: docs
sectionnumber: 1
---

## Windows Vagrant Setup

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


### Prerequisites

* VirtualBox 6 and higher requires 64-bit Windows.


### Connectivity Details

Windows Vagrant setup provides a local
CentOS virtual machine running under [VirtualBox][virtualbox] with the
following IP addresses and credentials.

```yaml
control: 192.168.122.60

user: vagrant
password: vagrant
```

On Windows ensure VirtualBox and Vagrant are installed.
The easiest way is to use [Chocolatey][chocolatey] to install
both of them.

In an **administrative powershell console** execute the following
commands:

```bash
# install chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force;
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# installl VirtualBox and Vagrant
choco install virtualbox vagrant

```

Open a new PowerShell console with your login account privileges
and execute the following commands.

```bash
# create directory and download Vagrantfile
mkdir prometheus-labs
cd prometheus-labs
iwr -OutFile Vagrantfile https://raw.githubusercontent.com/puzzle/prometheus-labs/master/Vagrantfile

# setup vm's
vagrant up
```

#### Techlab Shutdown

```bash
cd prometheus-labs

# shutdown all vm's
vagrant destroy -f
```
[virtualbox]: https://www.virtualbox.org/
[chocolatey]: https://chocolatey.org/

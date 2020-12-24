---
title: "Windows Vagrant setup"
description: "Vagrant installation on Windows"
weight: 3
type: docs
sectionnumber: 1
---

## Windows Vagrant setup

To participate in the lab you can use any Linux server
of your choice. The labs are tailored for setup with
CentOS/RHEL hosts. Below are instructions on how to setup
the required host with [Vagrant][vagrant] on Linux.
Follow the step-by-step guide to bootstrap the techlab
environment on your OS of choice.


### Prerequisites

* VirtualBox 6 and later requires 64-bit Windows


### Connectivity Details

{{% alert title="Note" color="primary" %}}
The following passwords are not secure and intended only to
be used with local virtual machines not reachable from outside
of the virtualization host.
{{% /alert %}}

Windows Vagrant setup provides a local
CentOS virtual machine running in [VirtualBox][virtualbox] with the
following IP address and credentials.

```yaml
control: 192.168.122.60

user: vagrant
password: vagrant
```

On Windows ensure VirtualBox and Vagrant are installed.
The easiest way to install both of them is to use [Chocolatey][chocolatey].

Open an **administrative PowerShell console** to execute the following commands.

Install Chocolatey:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force;
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Installl VirtualBox and Vagrant:

```powershell
choco install virtualbox vagrant
```

Open a new PowerShell console with your login account privileges
and execute the following commands.

Create the working directory and download the Vagrantfile:

```powershell
mkdir prometheus-labs
cd prometheus-labs
iwr -OutFile Vagrantfile https://raw.githubusercontent.com/puzzle/prometheus-labs/main/Vagrantfile
```

Start the virtual machine:

```powershell
vagrant up
```

Access it:

```powershell
ssh vagrant@192.168.122.60
```


#### Shutdown

Switch to the working directory and stop the virtual machine:

```bash
cd prometheus-labs
vagrant destroy -f
```

[virtualbox]: https://www.virtualbox.org/
[chocolatey]: https://chocolatey.org/

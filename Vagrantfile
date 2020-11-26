# -*- mode: ruby -*-
# vi: set ft=ruby :
# ensure SSH password login
$script = <<-SCRIPT
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum update -y
yum install -y kubectl docker-ce wget vim git
systemctl enable docker
systemctl start docker
usermod -aG docker vagrant
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
SCRIPT
Vagrant.configure("2") do |config|
  config.vm.provider :libvirt do |v|
    v.qemu_use_session = false
    v.cpus = 2
    v.memory = 6144
  end
  config.vm.define "prometheus" do |prometheus|
    prometheus.vm.box = "centos/8"
    prometheus.vm.hostname = "prometheus"
    prometheus.vm.network "private_network", ip: "192.168.122.60"
    prometheus.vm.network "forwarded_port", guest: "9090", host: "9090"
    prometheus.vm.network "forwarded_port", guest: "9093", host: "9093"
    prometheus.vm.network "forwarded_port", guest: "3000", host: "3000"
    prometheus.vm.provision "shell",
      inline: $script
  end
end

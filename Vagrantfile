# -*- mode: ruby -*-
# vi: set ft=ruby :
# ensure SSH password login
$script = <<-SCRIPT
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
yum install -y kubectl docker-ce wget vim git psmisc wget vim psmisc java-11-openjdk-devel gcc gcc-c++ sqlite-devel ruby-devel redhat-rpm-config make
kubectl completion bash >/etc/bash_completion.d/kubectl
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
usermod -aG docker vagrant
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
gem install mailcatcher -v 0.7.1
cat <<EOF > /etc/systemd/system/mailcatcher.service
[Unit]
Description=MailCatcher Service
After=network.service

[Service]
Type=simple
ExecStart=/usr/local/bin/mailcatcher --foreground --ip 0.0.0.0

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl restart sshd
sudo systemctl daemon-reload
sudo systemctl enable mailcatcher
sudo systemctl start mailcatcher
sudo yum update libseccomp -y
runuser -l vagrant -c "/usr/local/bin/minikube start \
  --kubernetes-version=v1.19.0 \
  --memory=5g \
  --cpus=2 \
  --bootstrapper=kubeadm \
  --extra-config=kubelet.authentication-token-webhook=true \
  --extra-config=kubelet.authorization-mode=Webhook \
  --extra-config=scheduler.address=0.0.0.0 \
  --extra-config=controller-manager.address=0.0.0.0"
runuser -l vagrant -c "/usr/local/bin/minikube addons enable ingress"
runuser -l vagrant -c "/usr/local/bin/minikube addons disable metrics-server"
SCRIPT
Vagrant.configure("2") do |config|
  config.vm.provider :libvirt do |v|
    v.qemu_use_session = false
    v.cpus = 2
    v.memory = 8192
  end
  config.vm.define "prometheus" do |prometheus|
    prometheus.vm.box = "centos/8"
    prometheus.vm.hostname = "prometheus"
    prometheus.vm.network "private_network", ip: "192.168.122.60"
    prometheus.vm.network "forwarded_port", guest: "9090", host: "9090"
    prometheus.vm.network "forwarded_port", guest: "9093", host: "9093"
    prometheus.vm.network "forwarded_port", guest: "3000", host: "3000"
    prometheus.vm.network "forwarded_port", guest: "1080", host: "1080"
    prometheus.vm.network "forwarded_port", guest: "19090", host: "19090"
    prometheus.vm.network "forwarded_port", guest: "19093", host: "19093"
    prometheus.vm.network "forwarded_port", guest: "13000", host: "13000"
    prometheus.vm.provision "shell",
      inline: $script
  end
end

# -*- mode: ruby -*-
# vi: set ft=ruby :
# ensure SSH password login
$script = <<-SCRIPT
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
yum install -y wget vim psmisc java-11-openjdk-devel gcc gcc-c++ sqlite-devel ruby-devel redhat-rpm-config make
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
systemctl daemon-reload
systemctl enable mailcatcher
systemctl start mailcatcher
SCRIPT
Vagrant.configure("2") do |config|
  config.vm.provider :libvirt do |v|
    v.qemu_use_session = false
  end
  config.vm.define "prometheus" do |prometheus|
    prometheus.vm.box = "centos/8"
    prometheus.vm.hostname = "prometheus"
    prometheus.vm.network "private_network", ip: "192.168.122.60"
    prometheus.vm.network "forwarded_port", guest: "9090", host: "9090"
    prometheus.vm.network "forwarded_port", guest: "9093", host: "9093"
    prometheus.vm.network "forwarded_port", guest: "3000", host: "3000"
    prometheus.vm.network "forwarded_port", guest: "1080", host: "1080"
    prometheus.vm.provision "shell",
      inline: $script
  end
end

---
title: "6. Visualization"
weight: 1
sectionnumber: 1
---

## Install Grafana

Add Grafana Repo
```bash
sudo bash -c 'cat <<EOF > /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF'
```

Install Grafana

```bash
sudo yum install grafana
```

Start Grafana and enable the service

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Check if you can login to your Grafana instance <http://localhost:3000/> with:

| Username | Password |
|---       |---       |
| admin    | admin    |

{{% alert title="Note" color="primary" %}}
Change password at first login
{{% /alert %}}

## Useful links and guides

* [Prometheus Data Source](https://grafana.com/docs/grafana/latest/datasources/prometheus/)


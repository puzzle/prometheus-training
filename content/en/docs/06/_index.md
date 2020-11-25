---
title: "6. Visualization"
weight: 1
sectionnumber: 1
---

{{% alert title="Note" color="primary" %}}
Our Goal with this lab is to give you a brief overview, how you can visualize your Prometheus time series produced in the previous labs.
For a more detailed tutorial please refer to [Grafana Tutorial](https://grafana.com/tutorials/).
{{% /alert %}}

## Installation

Add Grafana repository
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
* [Grafana Dashboards](https://grafana.com/docs/grafana/latest/best-practices/best-practices-for-creating-dashboards/)
* [Grafana Provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/)

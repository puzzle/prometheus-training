---
title: "3. Alerting with Alertmanager"
weight: 1
sectionnumber: 1
---

## Installation

### Setup

Before we can define rules for alerting we must install Alertmanager by downloading and extracting the Alertmanager binary.

1. First we need to create a working directory where we will download and extract Alertmanager. Open a new terminal, navigate to your home directory and create a new directory called prometheus:

    ```bash
    mkdir ~/alertmanager
    cd ~/alertmanager
    ```

1. Next we will download Alertmanager:

    ```bash
    curl -L -O https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz
    ```

    {{% alert title="Note" color="primary" %}}
Binaries for other CPU architectures such as ARM or other operating systems (darwin, bsd and even windows) are available on the release page of Alertmanager: https://github.com/prometheus/alertmanager/releases
    {{% /alert %}}

1. Extract the archive

    ```bash
    tar fvxz alertmanager-0.21.0.linux-amd64.tar.gz
    ```

1. Examining the contents of the tarball

    If you check the output of the previous tar command you should see list of extracted files. We will now take a closer look at some of these files:

    * **alertmanager**

        this is the Alertmanager binary itself

    * **amtool**

        a useful tool which can be used for debugging, testing, silencing and other tasks related to Alertmanager

    * **alertmanager.yml**

        this is the configuration file of Alertmanager. More on that in the next section about [Configuration](#Configuration).


### Configuration

The configuration of Alertmanager is done using a YAML config file and cli flags. The Alertmanager tarball we downloaded earlier includes a very basic example of a Alertmanager configuration file:

`alertmanager.yml`

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

{{% alert title="Note" color="primary" %}}
For the moment we leave it with the default configuration and come back to it later in the course.
{{% /alert %}}

### Run Alertmanager

To run Alertmanager you can simply execute the binary `alertmanager` and tell it where it can find its configuration file:

1. Open a new terminal and navigate to the extracted Alertmanager folder:

    ```bash
    cd ~/alertmanager/alertmanager-0.21.0.linux-amd64
    ```

1. Start Alertmanager by executing the binary:

    ```bash
    ./alertmanager --config.file=alertmanager.yml &
    ```

1. You should now see Alertmanager starting up and the log line `msg=Listening address=:9093."`. To verify this open your browser and navigate to [http://127.1:9093](http://127.1:9093). You should now see the Alertmanager webinterface

Before going on, let's make some warm-up [labs for monitoring your Alertmanager](labs/31)

## Enable Alertmanager in Prometheus

The Alertmanager instance we installed before must be configured in Prometheus: Open `prometheus.yml`, add alertmanager (see below) and restart or reload Prometheus.

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093
```

## Alert rules in Prometheus

{{% alert title="Note" color="primary" %}}
TODO: Think about replace this by a real sample application.

To go on with the training we need a basic simulation app which exposes a metric endpoint. Open a second ssh session to your vagrant virtual machine and execute the following commands:

```bash
# installation is needed only once => TODO: Integrate in Vagrantfile
sudo yum install -y nc

while true ; do
  count=$((count + 1))
  echo -ne "HTTP/1.0 200 OK\r\n\r\nhttp_request_count{handler=\"/\",method=\"GET\",status=\"200\"} $count" | nc -l -p 8080
done
```

Examine the metrics of this service with `curl localhost:8080`.
Finally, the target must be registered in Prometheus (don't forget to reload or restart Prometheus):

```yaml
  - job_name: 'sample-app-A'
    static_configs:
    - targets: ['localhost:8080']
```

{{% /alert %}}

[Prometheus alert rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) are configured very similar to recording rules which you got to know [earlier in this training](/docs/02#recording-rules). The main difference is that the rule's expression contains a threshold (e.g. `query_expression >= 5`) and that an alert is sent to the Alertmanager in case the rule evaluation matches the threshold. An alert rule can be based on a recording rule or be a normal expression query.

## Receivers in Alertmanager

## Routing rules in Alertmanager

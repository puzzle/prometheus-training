---
title: "3 Alerting with Alertmanager"
weight: 1
sectionnumber: 1
---

## Notes (to be removed)

### Ideas for slides

* Some background information about alertmanager (Github project, Architecture etc)
* Alertmanager can be engineered as cluster
* Other stuff which is out of scope in the basic training?
* What is Deduplication?

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
    ./alertmanager --config.file=alertmanager.yml
    ```

1. You should now see Alertmanager starting up and the log line `msg=Listening address=:9093."`. To verify this open your browser and navigate to [http://127.1:9093](http://127.1:9093). You should now see the Alertmanager webinterface

Before going on, let's make some warm-up [labs for monitoring your Alertmanager](labs/31)

## Enable Alertmanager in Prometheus

Prometheus doesn't know about Alertmanager, so far. In other words, Prometheus can't yet send any alerts so it makes
sense to enable Alertmanager before defining any alert rules.

Open `prometheus.yml`, enable alertmanager (see below) and restart or reload Prometheus.

```
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - alertmanager:9093
```

## Define alert rules in Prometheus


## Configure routing rules and receivers in Alertmanager

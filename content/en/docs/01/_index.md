---
title: "1 Setting up Prometheus"
weight: 1
sectionnumber: 1
---

## Installation

### Setup

Let's beginn with the installation of Prometheus by downloading and extracting the Prometheus binary.

1. At First we need to create a working directory where we will download and extract Prometheus. Open a new terminal, navigate to your home directory and create a new directory called prometheus:

    ```bash
    mkdir ~/prometheus
    cd ~/prometheus
    ```


1. Next we will download Prometheus:

    ```bash
    curl -L -O https://github.com/prometheus/prometheus/releases/download/v2.22.2/prometheus-2.22.2.linux-amd64.tar.gz
    ```

    {{% alert title="Note" color="primary" %}}
Binaries for other CPU architectures such as ARM or other operating systems (darwin, bsd and even windows) are available on the release page of Prometheus: https://github.com/prometheus/prometheus/releases
    {{% /alert %}}

1. Extract the archive

    ```bash
    tar fvxz prometheus-2.22.2.linux-amd64.tar.gz
    ```

1. Examining the contents of the tarball

    If you check the output of the previous tar command you should see list of extracted files. We will now take a closer look at some of these files:

    * **prometheus**

        this is the Prometheus binary itself

    * **promtool**

        a useful tool which can be used for debugging and querying Prometheus

    * **prometheus.yml**

        this is the configuration file of Prometheus. More on that in the next section (TODO: section reference)


### Configuration

The configuration of Prometheus is done using a YAML config file and cli flags. The Prometheus tarball we downloaded earlier includes a very basic example of a Prometheus configuration file:

`prometheus.yml`

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```

Let's take a look at two important configuration options:

* `scrape_interval`: Prometheus is a pull based monitoring system which means it will reach out to the configured targets and collects the metrics form them (instead of a push based approach where the targets will push their metrics to the monitoring server). The option `scrape_interval` defines the interval at which Prometheus will collect the metrics from each target.

* `scrape_configs`: This block defines which targets Prometheus will scrape. In the configuration above only a single target (the Prometheus server itself at `localhost:9090`) is configured. Check out the [Targets section](targets) for a detailed explanation.

{{% alert title="Note" color="primary" %}}
We will learn more about the other configuration options (`evaluation_interval`, `alerting` and `rule_files`) later in this course.
{{% /alert %}}

### Run Prometheus

To run Prometheus you can simply exe cute the binary `prometheus` and tell it where it can find it's configuration file:

1. Open a new terminal and navigate to the extracted Prometheus folder:

    ```bash
    cd ~/prometheus/prometheus-2.22.2.linux-amd64
    ```
1. Start Prometheus by executing the binary:

    ```bash
    ./prometheus --config.file=prometheus.yml
    ```
1. You should now see Prometheus starting up and the log line `msg="Server is ready to receive web requests."`. To verify this open your browser and navigate to [http://127.1:9090](http://127.1:9090). You should now see the Prometheus webinterface


## Targets

Since Prometheus is a pull based monitoring system the Prometheus server maintains a set of targets to scrape. This set can be configured using the `scrape_config` option in the Prometheus configuration file. The `scrape_config` consists of a list of jobs defining the targets as well as additional parameters (path, port, authentication etc.) required to scrape these targets.

{{% alert title="Note" color="secondary" %}}
Each job definition must at least consist of a `job_name` and a target configuration (e.g. `static_configs`).  Check the [Prometheus Docs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config) for the list of all available options in the `scrape_config`
{{% /alert %}}

There are two basic types of target configurations:

### static configuration

In this case the Prometheus configuration file contains a static list of targets. In order to make changes to the list you need to change the configuration file. We used this type of configuration in the previous section to scrape the metrics of the Prometheus server:

```yaml
scrape_configs:
  - job_name: 'prometheus' # this is a minimal example of a job definition containing the job_name and a target configuration
    static_configs:
    - targets:
      - server1:8080
      - server2:8080
```

### dynamic configuration

In addition to the static target configuration Prometheus provides many different ways to dynamically add/remove targets. There are builtin service discovery mechanisms for cloud providers such as AWS, GCP, Hetzner and many more. In addition there are more versatile discovery mechanisms available which allow you to implement Prometheus in your environment (e.g. DNS service discovery or file service discovery).
Let's take a look at an example of a file service discovery configuration:

```yaml
scrape_configs:
  - job_name: example_file_sd
    file_sd_configs:
    - files:
      - /etc/prometheus/file_sd/targets.yml
```
In this example Prometheus will lookup a list of targets in the file `/etc/prometheus/file_sd/targets.yml`. Prometheus will also pickup changes in the file automatically (without reloading) and adjust the list of targets accordingly.

## Advanced

<details><summary>Relabeling</summary>

### Relabeling

Relabeling in Prometheus can be used to perform numerous tasks using regular expressions such as:

* adding, modifying or removing labels to/from metrics or alerts
* filter metrics based on labels
* enable horizontal scaling of Prometheus by using hashmod relabeling

It is a very powerful part of the Prometheus configuration but it can also get quite complex and confusing. Thus we will only take a look at some basic / simple examples.

There are four types of relabeling:

* `relabel_configs` (target relabeling)

    Defined in the job definition of a `scrape_config`, used to relabel targets. This is used to configure scraping of multi-target exporter (e.g. blackbox_exporter or snmp_exporter) where one single exporter instance is used to scrape multiple targets. Check out The [Prometheus Docs](https://prometheus.io/docs/guides/multi-target-exporter/#querying-multi-target-exporters-with-prometheus) for a detailed explanation and example configurations of  `relabel_configs`.

* `metric_relabel_configs` (metrics relabeling)

    Metrics relabeling is applied to scraped samples right before ingestion. It allows to add / modify or drop labels or even drop entire samples if they match certain criteria.

* `alert_relabel_configs` (alert relabeling)
    Similar to `metric_relabel_configs` but applies to outgoing alerts

* `write_relabel_configs` (remote write relabeling)
    Similar to `metric_relabel_configs` but applies to `remote_write` configurations


</details>

---
title: "1. Setting up Prometheus"
weight: 1
sectionnumber: 1
---

## Installation

### Setup

Let's begin with the installation of Prometheus by downloading and extracting the Prometheus binary.

1. Open a new terminal, navigate to your home directory and create the directories `work` and `downloads`:

    ```bash
    mkdir ~/{work,downloads}
    cd ~/downloads
    ```


1. Download Prometheus:

    ```bash
    curl -L -O https://github.com/prometheus/prometheus/releases/download/v2.22.2/prometheus-2.22.2.linux-amd64.tar.gz
    ```

    {{% alert title="Note" color="primary" %}}
Binaries for other CPU architectures such as ARM or other operating systems (e.g., Darwin, BSD, Windows) are available on the release page of Prometheus: <https://github.com/prometheus/prometheus/releases>
    {{% /alert %}}

1. Extract the archive to the work folder:

    ```bash
    tar fvxz prometheus-2.22.2.linux-amd64.tar.gz -C ~/work
    ```

1. Examine the contents of the tarball:

    Check the output of the previous tar command. You should see a list of extracted files. We will now take a closer look at some of these files:

    * `prometheus`

       This is the Prometheus binary.

    * `promtool`

       This is a useful tool which can be used for debugging and querying Prometheus.

    * `prometheus.yml`

       This is the configuration file of Prometheus. More on that in the next section.


### Configuration

The configuration of Prometheus is done using a YAML config file and CLI flags. The Prometheus tarball we downloaded earlier includes a very basic example of a Prometheus configuration file:

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

* `scrape_interval`: Prometheus is a pull-based monitoring system which means it will reach out to the configured targets and collect the metrics from them (instead of a push-based approach where the targets will push their metrics to the monitoring server). The option `scrape_interval` defines the interval at which Prometheus will collect the metrics for each target.

* `scrape_configs`: This block defines which targets Prometheus will scrape. In the configuration above, only a single target (the Prometheus server itself at `localhost:9090`) is configured. Check out the [targets](#targets) section below for a detailed explanation.

{{% alert title="Note" color="primary" %}}
We will learn more about other configuration options (`evaluation_interval`, `alerting`, and `rule_files`) later in this training.
{{% /alert %}}

### Run Prometheus

{{% alert title="Note" color="primary" %}}
We will use Unix job control to run the binary. By adding the ampersand symbol (`&`) at the end of a command, the shell will put the command into the background. You can then use the command `jobs` to list all jobs currently running in the background of this shell and bring the jobs to the foreground by running `%1` (job number 1), `%2` (job number 2), etc. Please note that if you close a shell with background jobs, all these jobs will terminate.
You can use tools like `tmux`, `screen`, `nohup` or `disown` to keep jobs running even if you close the shell.
{{% /alert %}}

To run Prometheus, you can simply execute the `prometheus` binary and define where it can find its configuration file:

1. Open a new terminal and navigate to the extracted Prometheus folder:

    ```bash
    cd ~/work/prometheus-2.22.2.linux-amd64
    ```
1. Start Prometheus by executing the binary:

    ```bash
    ./prometheus --config.file=prometheus.yml &
    ```
1. You should now see Prometheus starting up with the log line `msg="Server is ready to receive web requests."`. To verify this, open your browser and navigate to <http://localhost:9090>. You should now see the Prometheus web UI.

{{% alert title="Note" color="primary" %}}
If you use the provided Vagrant setup then ports 9090 (Prometheus), 9093 (Alertmanager), and 3000 (Grafana) are forwarded to the VM where Prometheus is running.
Check out the Vagrantfile for details.

If you got another VM you may need to change `localhost` with the IP-Adress of your machine.
{{% /alert %}}


## Targets

Since Prometheus is a pull-based monitoring system, the Prometheus server maintains a set of targets to scrape. This set can be configured using the `scrape_configs` option in the Prometheus configuration file. The `scrape_configs` consist of a list of jobs defining the targets as well as additional parameters (path, port, authentication, etc.) which are required to scrape these targets.

{{% alert title="Note" color="primary" %}}
Each job definition must at least consist of a `job_name` and a target configuration (i.e., `static_configs`). Check the [Prometheus docs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config) for the list of all available options in the `scrape_config`.
{{% /alert %}}

There are two basic types of target configurations:

### Static configuration

In this case, the Prometheus configuration file contains a static list of targets. In order to make changes to the list, you need to change the configuration file. We used this type of configuration in the previous section to scrape the metrics of the Prometheus server:

```yaml
scrape_configs:
  - job_name: 'prometheus' # this is a minimal example of a job definition containing the job_name and a target configuration
    static_configs:
    - targets:
      - server1:8080
      - server2:8080
```

### Dynamic configuration

In addition to the static target configuration, Prometheus provides many ways to dynamically add/remove targets. There are builtin service discovery mechanisms for cloud providers such as AWS, GCP, Hetzner, and many more. In addition, there are more versatile discovery mechanisms available which allow you to implement Prometheus in your environment (e.g., DNS service discovery or file service discovery).
Let's take a look at an example of a file service discovery configuration:

```yaml
scrape_configs:
  - job_name: example_file_sd
    file_sd_configs:
    - files:
      - /etc/prometheus/file_sd/targets.yml
```
In this example, Prometheus will lookup a list of targets in the file `/etc/prometheus/file_sd/targets.yml`. Prometheus will also pickup changes in the file automatically (without reloading) and adjust the list of targets accordingly.


## Relabeling (advanced)

Relabeling in Prometheus can be used to perform numerous tasks using regular expressions, such as

* adding, modifying or removing labels to/from metrics or alerts,
* filtering metrics based on labels, or
* enabling horizontal scaling of Prometheus by using `hashmod` relabeling.

It is a very powerful part of the Prometheus configuration, but it can also get quite complex and confusing. Thus, we will only take a look at some basic/simple examples.

There are four types of relabelings:

* `relabel_configs` (target relabeling)

  Target relabeling is defined in the job definition of a `scrape_config`. This is used to configure scraping of a multi-target exporter (e.g., `blackbox_exporter` or `snmp_exporter`) where one single exporter instance is used to scrape multiple targets. Check out the [Prometheus docs](https://prometheus.io/docs/guides/multi-target-exporter/#querying-multi-target-exporters-with-prometheus) for a detailed explanation and example configurations of `relabel_configs`.

* `metric_relabel_configs` (metrics relabeling)

  Metrics relabeling is applied to scraped samples right before ingestion. It allows adding, modifying, or dropping labels or even dropping entire samples if they match certain criteria.

* `alert_relabel_configs` (alert relabeling)

  Alert relabeling is similar to `metric_relabel_configs`, but applies to outgoing alerts.

* `write_relabel_configs` (remote write relabeling)

  Remote write relabeling is similar to `metric_relabel_configs`, but applies to `remote_write` configurations.

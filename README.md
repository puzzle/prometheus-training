# Prometheus Basics Training

Interactive Prometheus Basics Training

## Geplante Inhalte

### Basics

* [ ] Hintergrundinfos zu Prometheus [#1](/../../issues/1)
* [ ] Architektur und Konzepte [#2](/../../issues/2)
* [ ] Vergleich zu anderen Monitoring-Lösungen [#3](/../../issues/3)
* [ ] Installation [#4](/../../issues/4)
* [ ] Targets [#5](/../../issues/5)
* [ ] Metrics [#6](/../../issues/6)
* [ ] Queries, PromQL [#7](/../../issues/7)
* [ ] Alerts [#8](/../../issues/8)
* [ ] Diverse Arten von Exportern [#9](/../../issues/9)
* [ ] Black Box Monitoring (Pushgateway) [#10](/../../issues/10)
* [ ] Exporters und Integration [#11](/../../issues/11)
* [ ] Metriken exploren [#12](/../../issues/12)
* [ ] Visualisierung [#13](/../../issues/13)
* [ ] Instrumentalisierung, Client Libraries [#14](/../../issues/14)
* [ ] Long Term Storage [#15](/../../issues/15)
* [ ] Recording Rules [#16](/../../issues/16)

### Best Practices

Können Bestandteil der jeweilgen Themen/Modulen sein, z.B.:

* [ ] Retention, Scraping intervall usw.
* [ ] Label Values, Labels, Relabeling
* [ ] Histogram vs Summary

### Infrastruktur Themen (Zielgruppe Systems Engineers)

* [ ] Configmanagement => Ansible?
* [ ] Backup
* [ ] Skalierung
  * Federation / Remote Write / Remote Read / Cortex / Thanos / VictoriaMetrics
* [ ] Monitoring von Appliances, Network Devices
* [ ] High Availability, Disaster Recovery

### Prometheus im Container Umfeld

* [ ] Prometheus/Grafana Operator
* [ ] Plattform Metriken
* [ ] Kubestate
* [ ] cadvisor
* [ ] Exporter als Sidecar
* [ ] Service Discovery
* [ ] Service Monitors, resp. annotations, Alertmanager Config

### Prometheus für Devs

* [ ] Developer Workflows
* [ ] Application Monitoring, Code Instrumentieren, beispiele mit JAVA, Go, Python
* [ ] Quering, Beispiele, wie komme ich jetzt an die Daten ran (evtl. eher Basis Modul)


## Content Sections

The training content resides within the [content](content) directory.

The main part are the labs, which can be found at [content/en/docs](content/en/docs).


## Hugo

This site is built using the static page generator [Hugo](https://gohugo.io/).

The page uses the [docsy theme](https://github.com/google/docsy) which is included as a Git Submodule.

After cloning the main repo, you need to initialize the submodule like this:

```bash
git submodule update --init --recursive
```


## Build using Docker

Build the image:

```bash
docker build -t acend/prometheus-basics-training:latest .
```

Run it locally:

```bash
docker run -i -p 8080:8080 acend/prometheus-basics-training
```


### Using Buildah and Podman

Build the image:

```bash
buildah build-using-dockerfile -t acend/prometheus-basics-training:latest .
```

Run it locally with the following command. Beware that `--rmi` automatically removes the built image when the container stops, so you either have to rebuild it or remove the parameter from the command.

```bash
podman run --rm --rmi --interactive --publish 8080:8080 localhost/acend/prometheus-basics-training
```


## How to develop locally

To develop locally we don't want to rebuild the entire container image every time something changed, and it is also important to use the same hugo versions like in production.
We simply mount the working directory into a running container, where hugo is started in the server mode.

```bash
docker run --rm --interactive --publish 8080:8080 -v $(pwd):/opt/app/src -w /opt/app/src acend/hugo:<version-in-dockerfile> hugo server -p 8080 --bind 0.0.0.0
```


## Linting of Markdown content

Markdown files are linted with <https://github.com/DavidAnson/markdownlint>.
Custom rules are in `.markdownlint.json`.
There's a GitHub Action `.github/workflows/markdownlint.yaml` for CI.
For local checks, you can either use Visual Studio Code with the corresponding extension, or the command line like this:

```shell script
npm install
node_modules/.bin/markdownlint content
```


## Contributions


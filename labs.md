# Geplante Inhalte

## Setup

* [ ] Setup mit Vargrant

## Präsentation

* [ ] Hintergrundinfos zu Prometheus [#1](/../../issues/1)
* [ ] Architektur und Konzepte [#2](/../../issues/2)
* [ ] Vergleich zu anderen Monitoring-Lösungen [#3](/../../issues/3)

## Basics Labs

* [1] Installation [#4](/../../issues/4)
* [2] Targets [#5](/../../issues/5)
* [3] Metrics [#6](/../../issues/6)
* [3] Metriken exploren [#12](/../../issues/12)
* [3] Queries, PromQL [#7](/../../issues/7)
* [4] Alerts [#8](/../../issues/8)
* [4] Recording Rules [#16](/../../issues/16)
* [5] Diverse Arten von Exportern [#9](/../../issues/9)
* [5] Black Box Monitoring (Pushgateway) [#10](/../../issues/10)
* [5] Exporters und Integration [#11](/../../issues/11)
* [6] Visualisierung [#13](/../../issues/13)
* [7] Instrumentalisierung, Client Libraries [#14](/../../issues/14)
* [8] Long Term Storage [#15](/../../issues/15)

## Best Practices

Können Bestandteil der jeweilgen Themen/Modulen sein, z.B.:

* [1] Retention
* [2] Scrape interval
* [2] Label Values, Labels, Relabeling
* [?] Histogram vs Summary

## Infrastruktur Themen (Zielgruppe Systems Engineers)

* [ ] Configmanagement => Ansible?
* [ ] Backup
* [ ] Skalierung
  * Federation / Remote Write / Remote Read / Cortex / Thanos / VictoriaMetrics
* [ ] Monitoring von Appliances, Network Devices
* [ ] High Availability, Disaster Recovery

## Prometheus im Container Umfeld

* [ ] Prometheus/Grafana Operator
* [ ] Plattform Metriken
* [ ] Kubestate
* [ ] cadvisor
* [ ] Exporter als Sidecar
* [ ] Service Discovery
* [ ] Service Monitors, resp. annotations, Alertmanager Config

## Prometheus für Devs

* [ ] Developer Workflows
* [ ] Application Monitoring, Code Instrumentieren, beispiele mit JAVA, Go, Python
* [ ] Quering, Beispiele, wie komme ich jetzt an die Daten ran (evtl. eher Basis Modul)

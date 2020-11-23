# Geplante Inhalte

## Setup

* [ ] Setup mit Vargrant

## Präsentation

* [ ] Hintergrundinfos zu Prometheus [#1](/../../issues/1)
* [ ] Architektur und Konzepte [#2](/../../issues/2)
* [ ] Vergleich zu anderen Monitoring-Lösungen [#3](/../../issues/3)

## Basics Labs

* [1] Installation [#4](/../../issues/4)
* [1] Targets [#5](/../../issues/5)
* [2] Metrics [#6](/../../issues/6)
* [2] Metriken exploren [#12](/../../issues/12)
* [2] Queries, PromQL [#7](/../../issues/7)
* [2] Recording Rules [#16](/../../issues/16)
* [3] Alerting rules + Alertmanager [#8](/../../issues/8)
* [4] Diverse Arten von Exportern [#9](/../../issues/9)
* [4] Black Box Monitoring (Pushgateway) [#10](/../../issues/10)
* [4] Exporters und Integration [#11](/../../issues/11)
* [5] Instrumentalisierung, Client Libraries [#14](/../../issues/14)
* [6] Visualisierung [#13](/../../issues/13)
* [7] Long Term Storage [#15](/../../issues/15)

## Best Practices

Können Bestandteil der jeweilgen Themen/Modulen sein, z.B.:

* [1] Retention
* [1] Scrape interval
* [2] Label Values, Labels (Relabeling -> advanced)
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

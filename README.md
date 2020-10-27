# Prometheus Labs

Interactive Prometheus Learning Labs

## Geplante Inhalte

### Basics

- [ ] Hintergrundinfos zu Prometheus
- [ ] Architektur und Konzepte
- [ ] Vergleich zu anderen Monitoring-Lösungen
- [ ] Installation (zu klären: Abhängig von Zielgruppe, oder Kursausprägung, Container oder Systemmonitoring)
- [ ] Targets
- [ ] Metric Types
- [ ] OpenMetrics (https://openmetrics.io/)
- [ ] Queries, PromQL
- [ ] Alerts
- [ ] Diverse Arten von Exportern (Blackbox, Push Gateway, Node Exporter, …)
- [ ] Black Box Monitoring (Pushgateway)
- [ ] Exporters und Integration
- [ ] Metriken exploren: Nodeexporter, Prometheus selber, Alertmanager Metriken
- [ ] Visualisierung
  - Dashboards mit Grafana
  - Prometheus-UI
- [ ] Instrumentalisierung, Client Libraries
- [ ] Long Term Storage
- [ ] Recording Rules

### Best Practices

Können Bestandteil der jeweilgen Themen/Modulen sein, z.B.:

- [ ] Retention, Scraping intervall usw.
- [ ] Label Values, Labels, Relabeling
- [ ] Histogram vs Summary

### Infrastruktur Themen (Zielgruppe Systems Engineers)

- [ ] Configmanagement => Ansible?
- [ ] Backup
- [ ] Skalierung
  - Federation / Remote Write / Remote Read / Cortex / Thanos / VictoriaMetrics
- [ ] Monitoring von Appliances, Network Devices
- [ ] High Availability, Disaster Recovery

### Prometheus im Container Umfeld

- [ ] Prometheus/Grafana Operator
- [ ] Plattform Metriken 
- [ ] Kubestate
- [ ] cadvisor
- [ ] Exporter als Sidecar
- [ ] Service Discovery 
- [ ] Service Monitors, resp. annotations, Alertmanager Config

### Prometheus für Devs

- [ ] Developer Workflows
- [ ] Application Monitoring, Code Instrumentieren, beispiele mit JAVA, Go, Python
- [ ] Quering, Beispiele, wie komme ich jetzt an die Daten ran (evtl. eher Basis Modul)

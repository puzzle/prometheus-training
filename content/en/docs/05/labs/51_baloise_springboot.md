---
title: "5.1 Tasks: Instrumenting"
weight: 2
sectionnumber: 5.1
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1: Spring Boot Example Instrumentation

Using the [micrometer metrics facade](https://spring.io/blog/2018/03/16/micrometer-spring-boot-2-s-new-application-metrics-collector) in Spring Boot Applications lets us collect all sort of metrics within a Spring Boot application. Those metrics can be exported for Prometheus to scrape by a few additional dependencies and configuration.

Let's have a deeper look at how the instrumentation of a Spring Boot application works. For that we can use the prometheus-training-spring-boot-example application located at https://github.com/acend/prometheus-training-spring-boot-example. To make the application collect metrics and provide a Prometheus endpoint we now need to simply add the following two dependencies in the `pom.xml` file, where it says `<!-- Add Dependencies here-->`:

{{% alert title="Note" color="primary" %}}
For your convenience, the changes mentioned below are already implemented in the `solution` subfolder of the git repository. You therefore do not have to make any changes in the code.
{{% /alert %}}

```xml
        ....
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <scope>runtime</scope>
        </dependency>
        ....
```

Additionally to those dependencies we also need to configure the metrics endpoints to be exposed.

This can be done in the file `src/main/resources/application.properties` by adding the following line:

```ini
management.endpoints.web.exposure.include=prometheus,health,info,metric
```

As mentioned above, these changes have already been implemented in the `solution` subfolder of the repository. A pre-built docker image is also available under <https://quay.io/repository/acend/prometheus-training-spring-boot-example?tab=tags>. In the next step we will deploy our application to our OpenShift Cluster:

```bash
{{% param cliToolName %}} -n [monitoring-namespace] apply -f https://raw.githubusercontent.com/puzzle/prometheus-training/main/content/en/docs/05/labs/springboot-example.yaml
```

{{% alert title="Note" color="primary" %}}
The command above will create a `Deployment`, a `Service` and a `ServiceMonitor` resource in our monitoring namespace. We will learn about `ServiceMonitors` later in labs 8. For now, we only need to know, that a `ServiceMonitor` resource will configure Prometheus targets based on the pods linked to the service.
{{% /alert %}}


### Task {{% param sectionnumber %}}.2: Metric names

Study the following metrics and decide if the metric name is ok

```promql
http_requests{handler="/", status="200"}

http_request_200_count{handler="/"}

go_memstats_heap_inuse_megabytes{instance="localhost:9090",job="prometheus"}

prometheus_build_info{branch="HEAD",goversion="go1.15.5",instance="localhost:9090",job="prometheus",revision="de1c1243f4dd66fbac3e8213e9a7bd8dbc9f38b2",version="2.32.1"}

prometheus_config_last_reload_success_timestamp{instance="localhost:9090",job="prometheus"}

prometheus_tsdb_lowest_timestamp_minutes{instance="localhost:9090",job="prometheus"}
```

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* The `_total` suffix should be appended, so `http_requests_total{handler="/", status="200"}` is better.

* There are two issues in `http_request_200_count{handler="/"}`: The `_count` suffix is foreseen for histograms, counters can be suffixed with `_total`. Second, status information should not be part of the metric name, a label `{status="200"}` is the better option.

* The base unit is `bytes` not `megabytes`, so `go_memstats_heap_inuse_bytes` is correct.

* Everything is ok with `prometheus_build_info` and its labels. It's a good practice to export such base information with a gauge.

* In `prometheus_config_last_reload_success_timestamp`, the base unit is missing, correct is `prometheus_config_last_reload_success_timestamp_seconds`.

* The base unit is `seconds` for timestamps, so `prometheus_tsdb_lowest_timestamp_seconds` is correct.

{{% /details %}}

### Task {{% param sectionnumber %}}.3: Metric names (optional)

What kind of risk do you have, when you see such a metric

```promql
http_requests_total{path="/etc/passwd", status="404"} 1
```

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

There is no potential security vulnerability from exposing the `/etc/passwd` path, which seems to be handled appropriately in this case: no password is revealed.

From a Prometheus point of view, however, there is the risk of a DDoS attack: An attacker could easily make requests to paths which obviously don't exist. As every request and therefore path is registered with a label, many new time series are created which could lead to a [cardinality explosion](https://www.robustperception.io/cardinality-is-key) and finally to out-of-memory errors.

It's hard to recover from that!

For this case, it's better just to count the 404 requests and to lookup the paths in the log files.

```promql
http_requests_total{status="404"} 15
```

{{% /details %}}

### Task {{% param sectionnumber %}}.4: Custom metric (optional)

In this lab you're going to create your own custom metric in the java Spring Boot application.

{{% alert title="Note" color="primary" %}}
This tasks requires that you have docker and git installed on your local machine.
This counter is just a simple example for the sake of this lab. Those kind of metrics are provided by the micrometer Prometheus Spring Boot integration out of the box.
{{% /alert %}}

First we need to clone the repository to our local machine:

```bash
git clone https://github.com/acend/prometheus-training-spring-boot-example && cd prometheus-training-spring-boot-example
```

and then configure the dependencies and `application.properties` as described in Task {{% param sectionnumber %}}.1.


Next, create a new CustomMetrics RestController class in your Spring Boot application `src/main/java/ch/acend/prometheustrainingspringbootexample/CustomMetricController.java`:

```java
package ch.acend.prometheustrainingspringbootexample;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;

@RestController
public class CustomMetricController {

    private final Counter myCounter;
    private final MeterRegistry meterRegistry;

    @Autowired
    public CustomMetricController(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.myCounter = meterRegistry.counter("my.prometheus.instrumentation.counter");
    }

    @GetMapping(value = "/api")
    public String getAll() {
        myCounter.increment();
        return "ok";
    }
}

```

We register our custom counter `myCounter` on the `MeterRegistry` in the constructor of the RestController.

Then we simply increase the counter every time the endpoint `/api` is hit. (just an example endpoint)

To build the application we will use the `Dockerfile` provided in the root folder of the repository.

```bash
docker build -t prometheus-training-spring-boot-example:local .
```

Start the Spring Boot application:

```bash
docker run --rm -p 8080:8080 prometheus-training-spring-boot-example:local
```

Let's create a couple of requests to our new endpoint, make sure to run those commands from a second terminal window, while the Spring Boot application is still running.

```bash
curl http://localhost:8080/api
```

Then verify the Prometheus metrics endpoint and look for a metric with the name `my_prometheus_instrumentation_counter_total`

```bash
curl http://localhost:8080/actuator/prometheus
```

Expected result:

```promql
...
# HELP my_prometheus_instrumentation_counter_total
# TYPE my_prometheus_instrumentation_counter_total counter
my_prometheus_instrumentation_counter_total 1.0
# HELP tomcat_sessions_rejected_sessions_total
# TYPE tomcat_sessions_rejected_sessions_total counter
tomcat_sessions_rejected_sessions_total 0.0
# HELP jvm_threads_peak_threads The peak live thread count since the Java virtual machine started or peak was reset
...
```

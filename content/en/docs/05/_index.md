---
title: "5. Instrumenting with client libraries"
weight: 1
sectionnumber: 1
---

While an exporter is an adapter for your service to adapt a service specific value into a metric in the Prometheus format, it is also possible to export metric data programmatically in your application code.

## Client libraries

The Prometheus project provides [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) which are either official or maintained by third-parties. There are libraries for major languages like Java, Go, Python, PHP, and .NET/C#.

Even if you don't plan to provide your own metrics, those libraries already export some basic metrics based on the language. For [Java](https://github.com/prometheus/client_java#included-collectors), default metrics about memory management (heap, garbage collection) and thread pools can be collected. The same applies to [Go](https://prometheus.io/docs/guides/go-application/).


### Spring Boot Example Instrumentation

Using the [micrometer metrics facade](https://spring.io/blog/2018/03/16/micrometer-spring-boot-2-s-new-application-metrics-collector) in Spring Boot Applications lets us collect all sort of metrics within a Spring Boot application. Those metrics can be exported for Prometheus to scrape by a few additional dependencies and configuration.

Let's have a deeper look at how the instrumentation of a Spring Boot application works.

Change to the downloads directory and clone the empty Spring Boot example git repository

```bash
cd ~/downloads
git clone https://github.com/acend/prometheus-training-spring-boot-example.git
```

Change into the freshly cloned git repository
```bash
cd prometheus-training-spring-boot-example
```

To make the application collect metrics and provide a Prometheus endpoint we now need to simply add the following two dependencies in the `pom.xml` file, where it says `<!-- Add Dependencies here-->`:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <scope>runtime</scope>
        </dependency>
```
Your `pom.xml` should look like this now.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>ch.acend</groupId>
    <artifactId>prometheus-training-spring-boot-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>prometheus-training-spring-boot-example</name>
    <description>This is the acend prometheus instrumentation example</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

Additionally to those dependencies we also need to configure the metrics endpoints to be exposed.

Add the following line to the file `src/main/resources/application.properties`

```
management.endpoints.web.exposure.include=prometheus,health,info,metric
```

Build the Spring Boot application:

```bash
./mvnw clean package
```

After the successful build, you can start the Application:

```bash
java -jar target/prometheus-training-spring-boot-example-0.0.1-SNAPSHOT.jar
```

Verify the metrics endpoint in a different terminal:

```bash
curl http://localhost:8080/actuator/prometheus
```

Expected result should look similar to

```
# HELP jvm_gc_memory_promoted_bytes_total Count of positive increases in the size of the old generation memory pool before GC to after GC
# TYPE jvm_gc_memory_promoted_bytes_total counter
jvm_gc_memory_promoted_bytes_total 1621496.0
# HELP tomcat_sessions_active_max_sessions  
# TYPE tomcat_sessions_active_max_sessions gauge
tomcat_sessions_active_max_sessions 0.0
...
```


## Specifications and conventions

There are some guidelines and best practices how to name your own metrics. Of course, the [specifications of the datamodel](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels) must be followed and applying the [best practices about naming](https://prometheus.io/docs/practices/naming/) is not a bad idea. All those guidelines and best practices are now officially specified in [openmetrics.io](https://openmetrics.io).

Following these principles is not (yet) a must, but it helps to understand and interpret your metrics.

You can check your metrics by using the following `promtool` command: `curl -s http://localhost:8080/metrics | promtool check metrics`

## Best practices

Though implementing a metric is an easy task from a technical point of view, it is not so easy to define what and how to measure. If you follow your existing [log statements](https://prometheus.io/docs/practices/instrumentation/#logging) and if you define an error counter to count all [errors and exceptions](https://prometheus.io/docs/practices/instrumentation/#failures), then you already have a good base to see the internal state of your application.

### The four golden signals

Another approach to define metrics is based on [the four golden signals](https://sre.google/sre-book/monitoring-distributed-systems/):

* Latency
* Traffic
* Errors
* Saturation

There are other methods like [RED](https://www.weave.works/blog/the-red-method-key-metrics-for-microservices-architecture/) or [USE](http://www.brendangregg.com/usemethod.html) that go into the same direction.

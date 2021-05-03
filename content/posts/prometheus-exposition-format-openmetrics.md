---
title: "OpenMetrics and Prometheus Exposition Format"
date: 2021-05-03T22:00:00+02:00
draft: false
tags: ["devops"]
description: "OpenMetrics is a CNCF project which aims to standardize the process of exposing metrics in cloud native apps"
---

OpenMetrics is a CNCF project which aims to standardize the process of exposing metrics in cloud native apps.
Specifications for OpenMetrics are public and can be read [here](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md)

## Prometheus Exposition Format
OpenMetrics is based on *Prometheus Exposition Format*. Which is a simple, text-based and human-readable data format with wide adoption in the community.

There were hundreds of integrations around this data format for Prometheus, even some other monitoring tools started reusing it.
However, having "Prometheus" in the name made it difficult to get support from competitors or other relevant parties.
OpenMetrics wants to change this by being a more vendor-neutral and _open_ data format. 
This may look like a strategic PR move, but it has the support of some of the largest organizations already. So looks like it has a great future.

### Data Model
Exposed metric data in OpenMetrics looks like this.
```
# TYPE acme_http_router_request_seconds summary
# UNIT acme_http_router_request_seconds seconds
# HELP acme_http_router_request_seconds Latency though all of ACME's HTTP request router.
acme_http_router_request_seconds_sum{path="/api/v1",method="GET"} 9036.32
acme_http_router_request_seconds_count{path="/api/v1",method="GET"} 807283.0
acme_http_router_request_seconds_created{path="/api/v1",method="GET"} 1605281325.0
acme_http_router_request_seconds_sum{path="/api/v2",method="POST"} 479.3
acme_http_router_request_seconds_count{path="/api/v2",method="POST"} 34.0
acme_http_router_request_seconds_created{path="/api/v2",method="POST"} 1605281325.0
# TYPE go_goroutines gauge
# HELP go_goroutines Number of goroutines that currently exist.
go_goroutines 69
# TYPE process_cpu_seconds counter
# UNIT process_cpu_seconds seconds
# HELP process_cpu_seconds Total user and system CPU time spent in seconds.
process_cpu_seconds_total 4.20072246e+06
# EOF
```
One of the best features of this format is the readability.

With *TYPE* we define the [metric type](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md#metric-types) and a label.
With *UNIT* we define the unit we are using in this MetricFamily.
*HELP* is a description of metric family for human consumption. Should be a short string so that it can be used as a tooltip.

## Metric types
### Counter
Counter is a metric type which counts discrete events. It must have a value called total and must be non-decreasing over time.
A counter can be used to count number of errors or requests, cpu time spent or some other cumulative data.

```
# TYPE process_cpu_seconds counter
# UNIT process_cpu_seconds seconds
# HELP process_cpu_seconds Total user and system CPU time spent in seconds.
process_cpu_seconds_total 4.20072246e+06
```

### Gauge
Gauge is one of the simplest metric types there is. It consists of a single value which can change over time. 
Contrary to counter, gauge can change its value at any given time and decrease in value.

```
# TYPE go_goroutines gauge
# HELP go_goroutines Number of goroutines that currently exist.
go_goroutines 69
```

### Histogram
Histograms are used to measure discrete events and split them into buckets. Common examples are request durations and I/O request sizes.
Histogram MetricPoints must have at least one bucket. count, created and sum values are optional.

```
# TYPE request_duration histogram
# HELP request_duration help
request_duration_bucket{le="0"} 0
request_duration_bucket{le="0.00000000001"} 0
request_duration_bucket{le="0.0000000001"} 0
request_duration_bucket{le="1e-04"} 0
request_duration_bucket{le="1.1e-4"} 0
request_duration_bucket{le="1.1e-3"} 0
request_duration_bucket{le="1.1e-2"} 0
request_duration_bucket{le="1"} 0
request_duration_bucket{le="1e+05"} 0
request_duration_bucket{le="10000000000"} 0
request_duration_bucket{le="100000000000.0"} 0
request_duration_bucket{le="+Inf"} 3
request_duration_count 3
request_duration_sum 2
# EOF
```

### Summary
Summaries are pretty similar to histograms, and they may be used when histograms are too expensive.
We can use sum, count and created values to create summaries.

```
# TYPE acme_http_router_request_seconds summary
# UNIT acme_http_router_request_seconds seconds
# HELP acme_http_router_request_seconds Latency though all of ACME's HTTP request router.
acme_http_router_request_seconds_sum{path="/api/v1",method="GET"} 9036.32
acme_http_router_request_seconds_count{path="/api/v1",method="GET"} 807283.0
acme_http_router_request_seconds_created{path="/api/v1",method="GET"} 1605281325.0
acme_http_router_request_seconds_sum{path="/api/v2",method="POST"} 479.3
acme_http_router_request_seconds_count{path="/api/v2",method="POST"} 34.0
acme_http_router_request_seconds_created{path="/api/v2",method="POST"} 1605281325.0
```

### Info
Info metrics are used to expose information which stays the same during the lifetime of a service.
Like build version, environment, region or service-name

## Metrics Endpoint
Implementers of this standard must expose metrics in OpenMetrics format a response to an HTTP Get request to a documented endpoint, 
which _should_ be named /metrics.
This endpoint will be consumed periodically, it must be able to respond with meaningful data for successive invocations. 

### Micrometer and Spring Actuator
My experience with exposing metrics(in Prometheus Exposition Format) to Datadog on a Spring Boot service 
was incredibly easy and satisfying with [Spring Boot Actuator and Micrometer](https://spring.io/blog/2018/03/16/micrometer-spring-boot-2-s-new-application-metrics-collector)

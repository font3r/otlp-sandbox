# Telemetry
Telemetry is the process of remotely collecting, transmitting, and analyzing data from devices or technical systems. 
It allows you to monitor operational parameters without being physically present at the location where the device is operating.

### How does it work?

- Measurement – measuring devices (e.g. sensors) record data (such as temperature, speed, voltage).
- Transmission – the data is transmitted (e.g. via GSM, Wi-Fi, satellite).
- Reception and analysis – the data is received by a system, stored, and analyzed.

### Instrumentation

In the context of telemetry, instrumentation refers to the hardware and software systems used to measure, monitor, 
and collect data from a physical system. It’s what makes telemetry possible by gathering the raw data that is then 
transmitted remotely for analysis.

- Instrumentation - the sensors, devices, and tools that collect data. In context of software it's the process 
  generating data at specific place, eg. instrumenting a library to generate usage data
- Telemetry - the process of sending that data somewhere else to be monitored or analyzed.

### Observability concepts
- Logs - timestamped text records of events
  - eg. 2025-08-02 14:33:01 - ERROR - Database connection failed
- Metrics - numeric measurements over time
  - eg. CPU/RAM usage = 72%, Requests per second = 120, Service avg/min/max latency
- Traces - records of how a request flows through multiple services
  - eg. single user request hits Service A → B → C, with timing and status at each step.

# OpenTelemetry (OTLP)
OpenTelemetry is a collection of APIs, SDKs, and tools. Use it to instrument, generate, collect, and export telemetry 
data (metrics, logs, and traces) to help you analyze your software’s performance and behavior.

OpenTelemetry is:
An observability framework and toolkit designed to facilitate the instrumentation, generation, collection, and export 
telemetry data (metrics, logs, and traces) to help you analyze your software’s performance and behavior. Open source, as well 
as vendor- and tool-agnostic, meaning that it can be used with a broad variety of observability backends, including open source 
tools like Jaeger and Prometheus, as well as commercial offerings. OpenTelemetry is not an observability backend itself.

![img.png](img.png)

OpenTelemetry is designed to be extensible. Some examples of how it can be extended include:

- Adding a receiver to the OpenTelemetry Collector to support telemetry data from a custom source
- Loading custom instrumentation libraries into an SDK
- Creating a distribution of an SDK or the Collector tailored to a specific use case (e.g. ELK OTLP collector)
- Creating a new exporter for a custom backend that doesn’t yet support the OpenTelemetry protocol (OTLP)
- Creating a custom propagator for a nonstandard context propagation format

### OTLP collector
The OpenTelemetry Collector offers a vendor-agnostic implementation of how to receive, process and export telemetry data. 
It removes the need to run, operate, and maintain multiple agents/collectors. This works with improved scalability and 
supports open source observability data formats (e.g. Jaeger, Prometheus, Fluent Bit, etc.) sending to one or more open 
source or commercial backends.

When to use a collector?

For most language specific instrumentation libraries you have exporters for popular backends and OTLP. You might wonder,
under what circumstances does one use a collector to send data, as opposed to having each service send directly to the backend?
For trying out and getting started with OpenTelemetry, sending your data directly to a backend is a great way to get 
value quickly. Also, in a development or small-scale environment you can get decent results without a collector.
However, in general we recommend using a collector alongside your service, since it allows your service to offload data 
quickly and the collector can take care of additional handling like retries, batching, encryption or even sensitive data filtering.
It is also easier to set up a collector than you might think: the default OTLP exporters in each language assume a local 
collector endpoint, so if you launch a collector it will automatically start receiving telemetry.

OTLP collector gives a flexibility to transform/process and export telemetry to other places, eg. we can export traces to one place,
metrics to second and logs to third or even duplicate them to other destinations

e.g. example config which listens on grpc/4137, http/4318 and exports metrics to prometheus, logs/traces to stdout
```yaml
# configure otlp collector ports/transfer protocol
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

# configure exporters (data destination) for later use
exporters:
  debug:
    verbosity: detailed
  prometheus:
    endpoint: ":9201"
    send_timestamps: true
    metric_expiration: 180m
    enable_open_metrics: true
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true

# configure where to export what type of data
service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [debug, otlp]
    metrics:
        receivers: [otlp]
        exporters: [debug, prometheus]
    logs:
      receivers: [otlp]
      exporters: [debug]
```
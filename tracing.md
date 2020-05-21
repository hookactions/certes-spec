# Tracing & Monitoring

Because of the distributed nature of this system, tracing and monitoring are of high importance. Not only for when things go wrong but also just to keep an eye on the health.

## Correlation IDs

Correlation IDs are a feature of Kafka and I see their use extending beyond Kafka itself. They are useful for seeing how some data has been sent throughout a distributed system. They are not a full replacement of other tracing and monitoring.

### Format

```
[ext.]<service>.<trace_id>
```

For example:

```
gw.1-67891233-abcdef012345678912345678
```

Where the trace id is defined as:

```
<version>.<time>.<id>
```

- `version`: The version number
- `time`: The epoch time, in seconds
- `id`: The trace identifier, a random string of 96 bits or 24 hexadecimal digits.

Each component in Certes has its own unique identifier as the `<service>` piece of the ID:

- `gw`: Event Gateway
- `eb`: Event Broker
- `api`: Master API
- `ec`: Event Consumer
- `reg`: Schema Registry

A single event may have more than one correlation ID such as, given event with an internal id of `abc` it might have the following correlation IDs:

```
ext.gh.1-67891233-abcdef012345678912345678
gw.1-67891233-abcdef012345678912345678
broker.1-67891233-abcdef012345678912345678
ec.1-67891233-abcdef012345678912345678
```

### Storage

The correlation ids should be stored in an order list or array. Order is important because it helps tracing where and how an Event has traveled. In most programming languages a regular list or array is sufficient, `set`s should be avoided.

## Tracing

In addition to Correlation IDs for tracing, I think it may be useful to provide an open way of tracing events and requests through the system.  I think that [The OpenTracing project](https://opentracing.io/) will provide these benefits, this would allow the traces to be sent to various tracing systems like:

- [Jaeger](https://www.jaegertracing.io/)
- [Datadog](https://www.datadoghq.com/)

Tracing, monitoring, and logging could all be handled by the newer [OpenTelemetry project](https://opentelemetry.io) which export to various formats.

## Monitoring

Each component will expose [Prometheus](https://prometheus.io/) metrics on the default port. The specific metrics are yet to be defined, but not necessarily important for this document.

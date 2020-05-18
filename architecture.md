# High-level Architecture

When thinking about the architechture of Certes, the best comparison I can make is a heterogeneous event bus. This means that the events in the event bus are not all the same type or structure. To explain this, let's look at what a homogeneous event bus looks like:

![Homogeneous event bus](/_media/certes_homogeneous_event_bus.png)

In this example, we have two event buses which contain a single type of event. There may be a single broker which sends the events into their proper bus or two separate brokers. This can be advantageous when you have separate APIs or event handlers for each different kind of event. Instead of having physically separate event buses, Certes uses virtual event buses. This means that there is a single queue (or event bus) and the consumer of this is responsible for fanning out the events to the proper APIs or event handlers.

![Heterogeneous event bus](/_media/certes_heterogeneous_event_bus.png)

This event bus could be thought of as a priority queue, where events of high value and importance are processed first (e.g. payments) and events of lower immediate importance are processed last (e.g. snack updates). Most producers of events may think that their events are high priority, but when every event is high priority none of them are. This priority is configured by the consumer of the events.

Now let's give a high level overview of each component. Each of these components have their own page which delve into further detail but if you just need a high-level explanation, read-on here.

![Certes components](/_media/certes_components.png)

## Event Gateway

The Event Gateway is the public facing interface into Certes. When I say "public facing", it may actually be available for the internet to see or it could be internal within a VPC network. This means that you could use Certes for internal events only if you wanted to, but generally you will want to receive events from other third-party sources so for our purposes "public" will mean "available to the internet".

This gateway can be thought of as a custom proxy like [Envoy](https://www.envoyproxy.io/) or [Nginx](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/). The implementation is discussed more in the [Event Gateway](/event-gateway) section.

This component is responsible for receiving incoming HTTP and GRPC requests and forwarding them to the proper components. Currently, there are three cases for forwarding:

1. A new incoming event is received over HTTP, it is forwarded to the [Event Broker](/event-broker)
1. An API request is received over HTTP, it is forwarded to the [Master API](/master-api#http)
1. An API request is received over GRPC, it is forwarded to the [Master API](/master-api#grpc)

Generally, there will be two ports available from this component:

1. `9600`: HTTP
1. `9601`: GRPC

## Event Broker

The Event Broker is responsible for handling the incoming event after it was received from the Event Gateway. It has the following responsibilities:

1. Validate the event format is valid via the Schema Registry
1. Store the event data via the Master API
1. Queue the event in the Message Queue

## Message Queue

The Message Queue is a priority queue whose underlying implementation is not entirely important. It may be SQS, Kafka, RabbitMQ, or even in-memory. The requirements for a queue are the following:

1. Priority based sorting
1. Ensure at-least once delivery

## Schema Registry

The Schema Registry is either the most complicated or least complicated component, depending on how you look at it. It has the following responsibilities:

1. Store and cache schemas in a file-system-like database (e.g. in-memory, boltdb, S3, file-system)
1. Validate incoming events against a schema

On paper, this seems very simple, however due to the nature of protobuf and a strongly typed language like Go, it may be complicated to dynamically load and validate a protobuf schema. Depending on the success or failure of using Go, this component may need to be written in another language like Rust (generics + metaprogramming) or Python (very dynamic and flexible).

## Master API

The Master API is the only component to interact with the state database directly. All other components should query and mutate data via this component. It will expose the API via GraphQL over HTTP and via GRPC.

- `9610`: HTTP
- `9611`: GRPC

This component is responsible for the following on the producer side of things:

1. Storing subscription data (i.e. where to send event data like URLs)
1. Storing event data (i.e. what data was sent to a subscriber)
1. Back-off information (i.e. how long to retry an event, how often to retry an event)
1. General configuration (i.e. feature flags, other global config)

From the subscriber side of things it is responsible for:

1. Storing subscription data (i.e. what third-parties are we subscribed to)
1. Back-off information (i.e. how long to retry an event, how often to retry an event)
1. Storing event data (i.e. what data was handled from a producer)
1. General configuration (i.e. feature flags, other global config)

## Event Consumer

The Event Consumer is a fairly straightforward component, it has the following responsibilities:

1. Handles new events from the Message Queue (either via polling or other subscription)
1. Verifies the HMAC signature of the event (retrieve HMAC secret from Master API)
1. Sends the event to the downstream consumer (only via HTTP right now, may include GRPC in the future)
1. Handles failure and requeues the event via the configured back-off rules

## Back-offs

In the previous Master API section, "back-offs" were briefly mentioned. Events **will** fail to send or be processed at some point and it is something we need to plan for. Traditionally, both the sender and receiver of a webhook may handle back-offs explicitly and in a complicated manner. Certes standardizes this in two ways:

1. The producer of events defines their back-off strategy when the consumer of events is unavailable to receive an event (e.g. Certes gateway is down, maintenance, etc.)
1. The subscriber of events their back-off strategy when the downstream API or event handler is unavailable (e.g. API server is down, maintenance, etc.)

The producer configuration is independent of the subscriber configuration and vice-versa. These configurations should be created via Master API or Management UI. There are some generic retry strategies included with Certes:

1. Logarithmic back-off configured via: maximum retries (absolute number) or maximum time to try (e.g. 7 days)
1. Linear back-off configured via: maximum retries (absolute number) or maximum time to try (e.g. 7 days)

These rules are set at a global level and may be configured per event source (e.g. GitHub) or per event type (e.g. GitHub push).

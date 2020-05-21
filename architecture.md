# High-level Architecture

When thinking about the architecture of Certes, the best comparison I can make is a heterogeneous event bus. This means that the events in the event bus are not all the same type or structure. To explain this, let's look at what a homogeneous event bus looks like:

![Homogeneous event bus](/_media/certes_homogeneous_event_bus.png)

In this example, we have two event buses which contain a single type of event. There may be a single broker which sends the events into their proper bus or two separate brokers. This can be advantageous when you have separate APIs or event handlers for each different kind of event. Instead of having physically separate event buses, Certes uses virtual event buses. This means that there is a single queue (or event bus) and the consumer of this is responsible for fanning out the events to the proper APIs or event handlers.

![Heterogeneous event bus](/_media/certes_heterogeneous_event_bus.png)

This event bus could be thought of as a priority queue, where events of high value and importance are processed first (e.g. payments) and events of lower immediate importance are processed last (e.g. snack updates). Most producers of events may think that their events are high priority, but when every event is high priority none of them are. This priority is configured by the consumer of the events.

Now let's give a high level overview of each component. Each of these components have their own page which delve into further detail but if you just need a high-level explanation, read-on here.

![Certes components](/_media/certes_components.png)

## Event Gateway

[summary](_media/event-gateway-summary.md ':include')

## Event Broker

[summary](_media/event-broker-summary.md ':include')

## Message Queue

[summary](_media/message-queue-summary.md ':include')

## Schema Registry

[summary](_media/schema-registry-summary.md ':include')

## Master API

[summary](_media/master-api-summary.md ':include')

## Event Consumer

[summary](_media/event-consumer-summary.md ':include')

## Back-offs

In the previous Master API section, "back-offs" were briefly mentioned. Events **will** fail to send or be processed at some point and it is something we need to plan for. Traditionally, both the sender and receiver of a webhook may handle back-offs explicitly and in a complicated manner. Certes standardizes this in two ways:

1. The producer of events defines their back-off strategy when the consumer of events is unavailable to receive an event (e.g. Certes gateway is down, maintenance, etc.)
1. The subscriber of events their back-off strategy when the downstream API or event handler is unavailable (e.g. API server is down, maintenance, etc.)

[summary](_media/backoffs-summary.md ':include')

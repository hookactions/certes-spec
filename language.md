# Ubiquitous Language

> By using the model-based language pervasively and not being satisfied until it flows, we approach a model that is complete and comprehensible, made up of simple elements that combine to express complex ideas.
>
> Domain experts should object to terms or structures that are awkward or inadequate to convey domain understanding; developers should watch for ambiguity or inconsistency that will trip up design.
>
> -- Eric Evans

It's important to define the terms for this project so that we are on the same page when discussing the different aspects of the system.

## Producer

Any entity, company, or person that:

1. defines Certes schemas;
1. allows an entity, company, or person to subscribe to a defined schema;
1. sends events to Subscribers.

A Producer may host their own Certes application or may use a third-party Certes provider.

## Subscriber

Any entity, company, or person that:

1. subscribes to Certes schema(s) defined by a Producer;
1. consumes messages sent by a Producer;

A Subscriber may host their own Certes application or may use a third-party Certes provider.

## Consumer

_See "[Subscriber](#subscriber)". These terms are used interchangeably and mean the same thing._

## Event

An Event is an instance of a specific version of a Schema that is sent from a Producer to a Subscriber.

## Subscription

When a Subscriber indicates through code (or more generally, the Master API) that they would like to receive Events from a Producer, a Subscription is created storing the necessary information to allow the Producer to send events to the Subscriber.

## Schema

A Schema defines the structure of an Event that a Producer will send to a Subscriber.

## Schema namespace

A Schema namespace is a grouping of Schema.

## URI

A URI is a "unique resource indicator" and we do not have a special definition other than the traditional meaning. Our specificity lies in the structure of the URI which is defined in the [URI section](/uri).

## Component

A Component is a self-contained application that is used within the greater Certes application. This component is interconnected with the other Certes components but is independent of the internal implementation of the other components.

## Back-off

When an Event is failed to be sent by the Producer to the Subscriber a Back-off period is set to avoid immediately re-sending the Event in case the Subscriber is temporarily unavailable or under heavy load. 

When an Event is failed to be consumed by the Subscriber, a Back-off period is set to avoid immediately re-sending the Event in case the downstream consumer is temporarily unavailable or under heavy load.

## Downstream consumer

The downstream consumer lives outside of Certes and is the application or code that is running to handle events that were previously subscribed to. Generally, this is an HTTP API or an application with an HTTP interface.

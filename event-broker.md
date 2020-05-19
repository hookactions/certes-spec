# Event Broker

[summary](_media/event-broker-summary.md ':include')

## Receiving a request

This component will expose a GRPC server on port `9620` with two main rpc's:

- `HandleIncomingEvent`
- `SendOutgoingEvent`

```protobuf
service EventBroker {
  rpc HandleIncomingMessage(...) returns (...) {}
  rpc SendOutgoingEvent(...) returns (...) {}
}
```

## HandleIncomingEvent

When a new event is received via HTTP by the [Event Gateway](/event-gateway) that will be transformed into a GRPC call to this component. Upon receiving this rpc, the following will happen:

1. Call the Schema Registry to validate the foreign event structure `grpc`
1. Store the event in the Master API `grpc`
1. Queue the event in the Message Queue

Upon storing the event in the Master API, a unique ID will be returned which should be queued. To limit the size of messages in the queue, only the ID should be queued. The Message Queue will deliver messages at-least once which means if the message is delivered twice we don't want to send the message to the subscriber twice. This deduplication will be explained more in the [Event Consumer](/event-consumer#deduplication).

## SendOutgoingEvent

For event producers, when they wish to send a new message to a subscriber, the `SendOutgoingEvent` rpc will be called. The flow is actually quite similar to `HandleIncomingEvent` with the major difference being the final destination. Instead of the Event Consumer sending the event to an internal service, it will send it to an external service/company who has subscribed to our events. In terms of data modeling, they could likely modeled the same way.

When receiving this rpc, the following will happen:

1. Call the Schema Registry to validate the event structure we defined `grpc`
1. Store the event in the Master API `grpc`
1. Queue the event in the Message Queue

Upon storing the event in the Master API, a unique ID will be returned which should be queued. To limit the size of messages in the queue, only the ID should be queued. The Message Queue will deliver messages at-least once which means if the message is delivered twice we don't want to send the message to the subscriber twice. This deduplication will be explained more in the [Event Consumer](/event-consumer#deduplication).

## Authentication

The traditional way of handling authentication with webhooks is using HMAC signatures of the body of the request. This component operates using GRPC which generally has two ways of handling authentication: SSL/TLS and Token-based authentication with Google. There are also libraries and examples of sending api keys via the message metadata.

There is a library called [`go-grpc-middleware`](https://github.com/grpc-ecosystem/go-grpc-middleware) which has many examples and packages of writing GRPC interceptors (or middleware).

To handle this, the `grpc-gateway` in the Event Gateway component will be responsible for verifying the integrity of incoming messages. Most of the time this will be a simple HMAC signature validation (the HMAC key should be retrieved from the Master API); other times this may require checking a header for a predefined value or some value from the JSON body. These special, non-HMAC, cases should only exist in the `community.certes.dev` events. Once a company commits to using Certes, HMAC signatures will automatically be generated and sent by the Event Consumer.

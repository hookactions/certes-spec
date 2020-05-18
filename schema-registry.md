# Schema Registry

[summary](_media/schema-registry-summary.md ':include')

## Overview

The Schema Registry will work similarly to a DNS server. The storage of foreign schemas by this service are ephemeral. The storage of schemas defined and stored for this Certes deployment, are most definitely not ephemeral. It works similar to a DNS server in the sense that when retrieving a schema for `community.certes.dev/github/repo/1/push` the protobuf schema will be stored locally in this components database. When fetching the schema via the Management UI or for validation purposes the local (cached) schema will be used. This will speed up things, but what about when a new backwards compatible version of the schema is released?

We talked about backwards compatibility in the [general schema overview](/schema#implementation). There are two events that are present with every Certes deployment which allow Certes deployments to communicate between themselves and update schemas as necessary. See "[Generic Events](#generic-events)" below.

## GRPC

```protobuf
service SchemaRegistry {
  rpc GetSchema(...) returns (...) {}
  rpc GetSchemaList(...) returns (...) {}

  rpc StoreSchema(...) returns (...) {}

  rpc ValidateEventForSchema(...) returns (...) {}
}
```

## Schema definition

## Schema storage

## Dynamic schema validation & usage

## Generic Events

It is sometimes useful for two or more Certes deployments to talk to each other, for example when a new schema is available or to check if the deployment is available with a simple "ping".

!> These are not finalized, just an example of what these events _might_ look like.

### Ping

When connecting to a new Certes deployment, a `Ping` rpc can be called to ensure that it is available and ready.

```protobuf
message Ping {}
message PingResponse {
  bool ok = 1;
  repeated Component components_unavailable = 2;
}
```

### VersionUpdate

When we update the `community.certes.dev/github/repo/1/push` schema, all subscribers to this events will receive a `VersionUpdate` event letting the Schema Registry know about the new version. Downstream subscribers can also listen to the `community.certes.dev/github/1/_version` event if needed. This will be sent for backwards compatible and incompatible changes. The schema registry should automatically replace backwards compatible schemas and only store the new backwards incompatible schema. Then when the consumer is ready they can subscribe to the new version. The new version can also be shown as available in the Management UI. 

```protobuf
message VersionUpdate {
  string old_schema_name = 1; // community.certes.dev/github/repo/1/push
  string new_schema_name = 2; // community.certes.dev/github/repo/1/push or community.certes.dev/github/repo/2/push
  bool backwards_compatible = 3;
}
message VersionUpdateResponse {}
```

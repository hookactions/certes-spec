# Event Schema

## Overview

At a high level, event schemas are defined used [proto3](https://developers.google.com/protocol-buffers/docs/proto3).

## Protobuf

Protocol buffers are language agnostic and code can be generated for just about any language based on a `.proto` file.
They are statically typed, which means extra type safety when also using a statically typed language such as Go. A downside to protobuf is that there is a little bit of a learning curve to define them as well as taking into consideration versioning (see the next section).

### Why not JSON?

A case could be made that using the Open API specification for defining the event schema would be adventageous because of the low learning curve to use JSON. My current opinion is that the benefits of a using a statically typed schema definition language such as protobuf outweigh the learning curve. A goal of Certes is to be safe and scalable, I don't believe JSON will ensure that. In fact, AWS Event Bridge uses JSON to define their schemas but they are extremely difficult to read and understand.

### Why not _X_?

If you would like to suggest a different schema definition language please create an issue in our GitHub respository. Keep the following in mind:

- Readability is key!
- Let's not reinvent the wheel
- Language agnostic?
- Current adoption & tooling?
- Learning curve?

## Versioning

Code changes over time, requirements change, so should our schemas be allowed to change. There are two types of changes to take into consideration: backwards compatible and backward incompatible.

### Backwards compatible

Backwards compatible changes mean that any changes to the schema will not break downstream consumers of the schema. Examples are:

- Adding a new field to a message
- Adding a new message type

### Backwards incompatible

Backwards incompatible changes mean that a change to the schema will break downstrewam consumers of the schema. Examples are:

- Removing a field
- Removing a message type
- Changing a field's type
- Adding a new enum member*
- Remving an enum member

*Adding a new enum member could be considered a backward compatible change. From the schema point of view, it is, but from a code point of view it may not be. Consider the following Go code:

```go
func myHandler(someEnum SomeEnum) {
  switch someEnum {
    case SomeEnumVal1: handleVal1()
    case SomeEnumVal2: handleVal2()
    case SomeEnumVal3: handleVal3()
    default: handleDefaultCase()
  }
}
```

If we're not careful, the default case handler could be to panic for unexpected values. Which could be valid in some cases where it's important what the enum members represent (i.e. payment states) and an unexpected member should not be handled. But in other cases, it might be ok because we only care about 2 out of the 10 enum members (i.e. event types). For now, this will be a backwards incompatible change, but this may _change_ in the future.

### Implementation

When ensuring subscriptions are active from code, the URI will contain a version number. In terms of SemVer, the following URI would be approximately equal to `^1.0.0` in SemVer: `community.certes.dev/github/1/push`. This means that any version updates such as `1.1.0` or `1.2.0` would still be sent to this subscriber because they are backwards compatible changes. When version `2.0.0` is released, those events would not be sent to the consumer unless they update their code to subscribe to `community.certes.dev/github/2/push`.

New versions can be uploaded via the [Management UI](#todo) or [API](#todo). They will automatically versioned and checked for backwards compatibility.

## Example

**NOTE:** this is not an active schema definition, is just shows what it _could_ look like.

```protobuf
// from: https://developer.calendly.com/docs/sample-webhook-data

syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "google/protobuf/any.proto";

enum EventKind {
    UNKNOWN = 0;
    INVITEE_CREATED = 1;
    INVITEE_CANCELED = 2;
}

message Webhook {
    EventKind event = 1;
    google.protobuf.Timestamp time = 2;
    Payload payload = 3;

    int64 calendly_hook_id = 4;
}

message Payload {
    EventType event_type = 1;
    Event event = 2;
    Invitee invitee = 3;
    repeated QuestionAnswer questions_and_answers = 4;
    map<string, string> questions_and_responses = 5;  // really want to limit the use of map
    Tracking tracking = 6;

    google.protobuf.Any old_event = 7; // ?
    google.protobuf.Any old_invitee = 8; // ?
    google.protobuf.Any new_event = 9; // ?
    google.protobuf.Any new_invitee = 10; // ?
}

message EventType {
    string uuid = 1;
    string kind = 2;
    string slug = 3;
    string name = 4;
    int32 duration = 5;
    EventOwner owner = 6;
}

message EventOwner {
    string type = 1;
    string uuid = 2;
}

message Event {
    string uuid = 1;
    repeated string assigned_to = 2;
    repeated ExtendedAssignedTo extended_assigned_to = 3;

    google.protobuf.Timestamp start_time = 4;
    string start_time_pretty = 5;

    google.protobuf.Timestamp end_time = 6;
    string end_time_pretty = 7;

    google.protobuf.Timestamp invitee_end_time = 8;
    string invitee_end_time_pretty = 9;

    google.protobuf.Timestamp created_at = 10;
    string location = 11;

    bool canceled = 12;
    string canceler_name = 13;
    string cancel_reason = 14;
    google.protobuf.Timestamp canceled_at = 15;
}

message ExtendedAssignedTo {
    string name = 1;
    string email = 2;
    bool primary = 3;
}

message Invitee {
    string uuid = 1;
    string first_name = 2;
    string last_name = 3;
    string name = 4;
    string email = 5;
    string text_reminder_number = 6;
    string timezone = 7;
    google.protobuf.Timestamp created_at = 8;
    bool is_reschedule = 9;
    repeated Payment payments = 10;
    bool canceled = 11;
    string canceler_name = 12;
    string cancel_reason = 13;
    google.protobuf.Timestamp canceled_at = 14;
}

message Payment {
    string id = 1;
    string provider = 2;
    double amount = 3;
    string currency = 4;
    string terms = 5;
    bool successful = 6;
}

message QuestionAnswer {
    string question = 1;
    string answer = 2;
}

message Tracking {
    string utm_campaign = 1;
    string utm_source = 2;
    string utm_medium = 3;
    string utm_content = 4;
    string utm_term = 5;
    string salesforce_uuid = 6;
}
```

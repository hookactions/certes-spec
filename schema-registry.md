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

There should be a user configurable limit to either the maximum number of schemas to cache or a limit of the total size. The interface to interacting with the storage might look like this:

```go
// File represents a file in the filesystem.
type File interface {
	io.Closer
	io.Reader
	io.ReaderAt
	io.Seeker
	io.Writer
	io.WriterAt

	Name() string
	Readdir(count int) ([]os.FileInfo, error)
	Readdirnames(n int) ([]string, error)
	Stat() (os.FileInfo, error)
	Sync() error
	Truncate(size int64) error
	WriteString(s string) (ret int, err error)
}

// Fs is the filesystem interface.
//
// Any simulated or real filesystem should implement this interface.
type Fs interface {
	// Create creates a file in the filesystem, returning the file and an
	// error, if any happens.
	Create(name string) (File, error)

	// Mkdir creates a directory in the filesystem, return an error if any
	// happens.
	Mkdir(name string, perm os.FileMode) error

	// MkdirAll creates a directory path and all parents that does not exist
	// yet.
	MkdirAll(path string, perm os.FileMode) error

	// Open opens a file, returning it or an error, if any happens.
	Open(name string) (File, error)

	// OpenFile opens a file using the given flags and the given mode.
	OpenFile(name string, flag int, perm os.FileMode) (File, error)

	// Remove removes a file identified by name, returning an error, if any
	// happens.
	Remove(name string) error

	// RemoveAll removes a directory path and any children it contains. It
	// does not fail if the path does not exist (return nil).
	RemoveAll(path string) error

	// Rename renames a file.
	Rename(oldname, newname string) error

	// Stat returns a FileInfo describing the named file, or an error, if any
	// happens.
	Stat(name string) (os.FileInfo, error)

	// The name of this FileSystem
	Name() string

	//Chmod changes the mode of the named file to mode.
	Chmod(name string, mode os.FileMode) error

	//Chtimes changes the access and modification times of the named file
	Chtimes(name string, atime time.Time, mtime time.Time) error
}
```

There could be various backends implemented for this, including:

- [bolt](https://github.com/etcd-io/bbolt)
- In-memory
- S3
- File-system

In fact, we could use the [afero](https://github.com/spf13/afero) golang library for this, even though we don't need _all_ of the functionality around chmod and such.

## Dynamic schema validation & usage

This is the hardest part of the whole Certes project I think.
We will need to use the [`google.protobuf.Any`](https://developers.google.com/protocol-buffers/docs/proto3#any) type. Let's take for example the Event Broker:

```protobuf
service EventBroker {
  rpc HandleIncomingEvent(IncomingEvent) returns (...) {};
}

message IncomingEvent {
  string schema = 1;  // community.certes.dev/github/repo/1/push
  repeated Header headers = 2;  // {"X-Foo": ["bar", "baz"]}
  google.protobuf.Any body = 3;
}

message Header {
  string name = 1;
  repeated string values = 2;
}
```

The Schema Registry might look like this:

```protobuf
service SchemaRegistry {
  rpc ValidateEventForSchema(ValidateEventSchemaRequest) returns (...) {};
}

message ValidateEventSchemaRequest {
  string schema = 1;
  google.protobuf.Any body = 3;
}
```

In the protobuf examples of `Any` you are able to unpack the `Any` field to a known type:

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

```go
package anything_test

//go:generate protoc --go_out=. anything.proto

import (
	"reflect"
	"testing"

	proto "github.com/golang/protobuf/proto"
	"github.com/golang/protobuf/ptypes"
	any "github.com/golang/protobuf/ptypes/any"
	"github.com/golang/protobuf/ptypes/timestamp"
	"github.com/pokstad/anything"
)

func TestAnything(t *testing.T) {
	t1 := &timestamp.Timestamp{
		Seconds: 5, // easy to verify
		Nanos:   6, // easy to verify
	}

	serialized, err := proto.Marshal(t1)
	if err != nil {
		t.Fatal("could not serialize timestamp")
	}

	// Blue was a great album by 3EB, before Cadgogan got kicked out
	// and Jenkins went full primadonna
	a := anything.AnythingForYou{
		Anything: &any.Any{
			TypeUrl: "example.com/yaddayaddayadda/" + proto.MessageName(t1),
			Value:   serialized,
		},
	}

	// marshal to simulate going on the wire:
	serializedA, err := proto.Marshal(&a)
	if err != nil {
		t.Fatal("could not serialize anything")
	}

	// unmarshal to simulate coming off the wire
	var a2 anything.AnythingForYou
	if err := proto.Unmarshal(serializedA, &a2); err != nil {
		t.Fatal("could not deserialize anything")
	}

	// unmarshal the timestamp
	var t2 timestamp.Timestamp
	if err := ptypes.UnmarshalAny(a2.Anything, &t2); err != nil {
		t.Fatalf("Could not unmarshal timestamp from anything field: %s", err)
	}

	// Verify the values are as expected
	if !reflect.DeepEqual(t1, &t2) {
		t.Fatalf("Values don't match up:\n %+v \n %+v", t1, t2)
	}
}
```

So theoretically we could `Unmarshal` to an `interface{}` but that doesn't validate that the schema is valid. This is why I'm thinking this component may need to be written in a dynamic language like python. This is the [example](https://developers.google.com/protocol-buffers/docs/reference/python-generated#any) code for `Unpack`ing an `Any` message. It seems like we may still need to pass in the correct protobuf descriptor, but with Python, we could just `exec(...)` the dynamically generated code for any new schemas. It's obviously much less safe, but I'm not sure how else to handle this with Go.

```python
any_message.Pack(message)
any_message.Unpack(message)
```

The reason this _could_ work with Python is that we could dynamically generate the Python code using `protoc` then call `exec(...)` on the result to have dynamic access to the message types.

```python
protoc_result = ...
data = {}
exec(protc_result, globals(), data)

message_type = data["GitHubPushMessage"]
any_message.Unpack(message_type)
```

There are many security implications around this which may leave this component vulnerable.

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

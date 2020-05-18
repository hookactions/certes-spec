The Schema Registry is either the most complicated or least complicated component, depending on how you look at it. It has the following responsibilities:

1. Store and cache schemas in a file-system-like database (e.g. in-memory, boltdb, S3, file-system)
1. Validate incoming events against a schema

On paper, this seems very simple, however due to the nature of protobuf and a strongly typed language like Go, it may be complicated to dynamically load and validate a protobuf schema. Depending on the success or failure of using Go, this component may need to be written in another language like Rust (generics + metaprogramming) or Python (very dynamic and flexible).

# Master API

[summary](_media/master-api-summary.md ':include')

## Available operations

For most of these operations they will just interact with the state database, however the schema related operations will interact with the Schema Registry.

1. Retrieve subscriptions
1. Create new subscriptions
1. List all received/sent events w/ trace information
1. Store new events
1. Update event state
1. Retrieve back-off configuration
1. Update back-off configuration
1. Retrieve global configuration (e.g. feature flags)
1. Update global configuration
1. Retrieve available schemas
1. Create new schemas
1. Add/update/remove users
1. Add/remove API keys

!> _TODO_: define input and output of these operations

## GraphQL vs REST

When creating the [Management UI](/management-ui) it will be nicer to work with GraphQL instead of REST. GraphQL also is type-safe which will allow us to write safer frontend and backend code. We do have the option of using another `grpc-gateway` to simply convert the GRPC endpoints to REST but I think the GraphQL schema will include more operations and types than we may want to define in GRPC.

## When to use GRPC or GraphQL?

Default to using GRPC, especially for backend based services and absolutely for all internal Certes components. GraphQL's main benefit is building frontend interfaces, so the most common use case will be the Management UI.

Another way to think about it is that GRPC will generally be used for writing and GraphQL for reading.

## Authentication

The Master API can be authenticated in a few different ways, the main differences come when talking about HTTP or GRPC.

### HTTP

When authenticating over HTTP a [JWT](https://jwt.io) will be used in the `Authorization` header like so:

```
Authorization: JWT xxx.xxx.xxx
```

The Master API will be responsible for creating new JWTs but other components may need to validate these tokens. The best approach is a public/private key pair so the components only needing to validate the token can do so with the public key. At this time `RS256` seems like the best signing algorithm; it is fast and secure.

### GRPC

As with the other components in Certes, the default handling of internal GRPC authentication is with SSL/TLS. If the GRPC methods are called from the Event Gateway then a JWT will be required in the GRPC request metadata. This JWT will be validated in the same manner as previously discussed in the HTTP section, the only difference is how it is generated. For GRPC use, a JWT may need to be long-lived and will be treated as an API key which can be provisioned in the Management UI.

### JWT data

The main value the JWT will have are the "roles" which are granted to the token. 

- `subscriptions:read`
- `subscriptions:write`
- `events:read`
- `events:write`
- `backoffs:read`
- `backoffs:write`
- `config:read`
- `config:write`
- `schemas:read`
- `schemas:write`

## Users

It will likely be useful to have some concept of users when interacting with the API. This can be useful when seeing who changed a configuration value last as well as just defining permissions of who has access to the API. Each internal component will have an implicit user assigned when interacting with the API via GRPC. When interacting with the API over GraphQL and the Management UI, an explicit user will be required and stored in the JWT along with the roles.

## Database

The default database will be Postgres because I've worked with it the most in the past. There should be an implementable interface for the database so other drivers could be used in the future. An in-memory database such as [QL](https://gitlab.com/cznic/ql) should also be implemented for easier testing.

!> _TODO_: define necessary models

There shouldn't be a need for a cache like Redis as long as we define proper indices.

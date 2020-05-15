# URIs

The URIs for Certes should be straightforward and easy to understand. There is somewhat of a chicken and egg problem when discussing the URIs because an understanding of the [architechture](#todo) of Certes is needed. It is worthwhile to have a high-level understanding of the architechture before reading on.

Now that you have a high-level understanding of the architechture, all URIs map to the Certes gateway. The gateway will then direct the request to the various internal components to handle. To start, let's think of these only in terms of HTTP, we'll come back to GRPC later.

Context is also important when discussing URIs. In one context, the URI `community.certes.dev/github/1/push` would specify the GitHub push event, but in another context it would just be gibberish. This is an attempt to keep implementation clean and simple. The tradeoff is that you need to keep these contexts in mind when you see a URI.

## Contexts

There are a few different contexts to keep in mind when working with Certes URIs. If the context of a URI is not clear, it should be assumed to be the "General" context.

### General (HTTP)

We are not reinventing the wheel for URIs. We are creating a new URI schema called `events://` which will default the URI to port `9600`. For example: `events://community.certes.dev` would be equal to `events://community.certes.dev:9600`.

Given no context or the specific general context, URIs will be taken at face value (this will make more sense when reading the other contexts).

#### Examples

1. Health Check: `events://community.certes.dev => community.certes.dev:9600`
1. Event Broker: `events://community.certes.dev/events => community.certes.dev:9600/events`
1. Master API: `events://community.certes.dev/api => community.certes.dev:9600/api`

These are the only 3 valid HTTP endpoints. Each of these have sub-endpoints not discussed here.

#### Usage

1. Initialization of sdk: `certes.Init("events://events.meetly.com")`
  - This specifies your Certes gateway. You can host this yourself or have it managed by a third-party. such as [HookActions](https://hookactions.com).
1. When adding new producers in the "[Management UI](#todo)": `events://community.certes.dev`
  - The Management UI will use this URI to discover the available events and their schema.
  - The Management UI will use this URI to determine how to authenticate with the given third-party.

### General (GRPC)

_TODO_

### Schema

### Event broker

### Master API

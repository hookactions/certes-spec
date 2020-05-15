# URIs

The URIs for Certes should be straightforward and easy to understand. There is somewhat of a chicken and egg problem when discussing the URIs because an understanding of the [architechture](#todo) of Certes is needed. It is worthwhile to have a high-level understanding of the architechture before reading on.

Now that you have a high-level understanding of the architechture, all URIs map to the Certes gateway. The gateway will then direct the request to the various internal components to handle. To start, let's think of these only in terms of HTTP, we'll come back to GRPC later.

Context is also important when discussing URIs. In one context, the URI `community.certes.dev/github/1/push` would specify the GitHub push schema, but in another context it would just be gibberish. This is an attempt to keep implementation clean and simple. The tradeoff is that you need to keep these contexts in mind when you see a URI.

## Contexts

There are a few different contexts to keep in mind when working with Certes URIs. If the context of a URI is not clear, it should be assumed to be the "General" context.

### General

### Schema

### Event broker

### Master API

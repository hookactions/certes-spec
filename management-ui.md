# Management UI

The Management UI is not an official component of Certes but it is worth mentioning in the spec. When working with complex systems, an UI is a very helpful piece of the puzzle to efficiently managing said system. An example of the official unofficial management UI could be compared to Flower from Celery. It gives some insight into the system and provides some very basic ways of interacting with the API.

This means that the default UI will likely just be a very simple, generic interface that is almost one to one with the Master API. It may even be automatically generated from the GraphQL schema.

## Other Certes Providers

I expect there to be other Certes providers for companies that are too small to host Certes themselves or wish to use something quick and easy for a proof of concept. The best similarity I can draw is that a Certes Provider is to Certes is what Cloudflare is to DNS.

There are many providers for DNS and they each provide advantages over the other. This is what Certes providers will be. They may have their own APIs that build on top of the Certes Master API to solve some use cases more efficiently. Let's give an example:

Entities:
- Meetly
- GitHub
- HookActions

Meetly is a small company who wants to receive events using Certes from GitHub. GitHub is a large company so they host their own instance of Certes.

Meetly uses HookActions, which is a Certes provider, to host a Certes instance and use the HookActions Certes Management UI. GitHub hosts their own instance of Certes to enable more complex scaling and configuration but uses HookActions as the Management UI.

What would the URIs map to?

- events.meetly.com &rarr; events.hookactions.com &rarr; _HookActions hosted Certes instance_
- mgmt.events.meetly.com &rarr; mgmt.events.hookactions.com &rarr; _HookActions management UI_
- events.github.com &rarr; _GitHub Certes instance_
- mgmt.events.github.com &rarr; mgmt.events.hookactions.com &rarr; _HookActions management UI_

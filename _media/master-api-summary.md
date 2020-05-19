The Master API is the only component to interact with the state database directly. All other components should query and mutate data via this component. It will expose the API via GraphQL over HTTP and via GRPC.

- `9610`: HTTP
- `9611`: GRPC

This component is responsible for the following on the producer side of things:

1. Storing subscription data (i.e. where to send event data like URLs)
1. Storing event data (i.e. what data was sent to a subscriber)
1. Back-off information (i.e. how long to retry an event, how often to retry an event)
1. General configuration (i.e. feature flags, other global config)

From the subscriber side of things it is responsible for:

1. Storing subscription data (i.e. what third-parties are we subscribed to)
1. Back-off information (i.e. how long to retry an event, how often to retry an event)
1. Storing event data (i.e. what data was handled from a producer)
1. General configuration (i.e. feature flags, other global config)

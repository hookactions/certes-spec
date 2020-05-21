The Event Consumer is a fairly straightforward component, it has the following responsibilities:

1. Handles new events from the Message Queue (either via polling or other subscription)
1. Generates the HMAC signature of the event (retrieve HMAC secret from Master API)
1. Sends the event to the downstream consumer (only via HTTP right now, may include GRPC in the future)
1. Handles failure and requeues the event via the configured back-off rules

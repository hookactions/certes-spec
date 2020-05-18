The Event Broker is responsible for handling the incoming event after it was received from the Event Gateway. It has the following responsibilities:

1. Validate the event format is valid via the Schema Registry
1. Store the event data via the Master API
1. Queue the event in the Message Queue

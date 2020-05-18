The Message Queue is a priority queue whose underlying implementation is not entirely important. It may be SQS, Kafka, RabbitMQ, or even in-memory. The requirements for a queue are the following:

1. Priority based sorting
1. Ensure at-least once delivery

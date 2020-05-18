# Message Queue

[summary](_media/message-queue-summary.md ':include')

> In computer science, a priority queue is an abstract data type similar to regular queue or stack data structure in which each element additionally has a "priority" associated with it. In a priority queue, an element with high priority is served before an element with low priority. In some implementations, if two elements have the same priority, they are served according to the order in which they were enqueued, while in other implementations, ordering of elements with the same priority is undefined.

> While priority queues are often implemented with heaps, they are conceptually distinct from heaps. A priority queue is a concept like "a list" or "a map"; just as a list can be implemented with a linked list or an array, a priority queue can be implemented with a heap or a variety of other methods such as an unordered array. 

See: [Priority queue from Wikipedia](https://en.wikipedia.org/wiki/Priority_queue)

Some queues have priority built-in (RabbitMQ) while others do not (SQS, Kafka). Generally for queues who do not natively support priority, the fallback is to create N queues for N priorities. As stated before, the priority is specified by the consumer/subscriber of the events **not** the producer.

Let's take a brief look at how some different queues may require their producers and consumers to work.

## In-memory

An in-memory message queue could be a simple GRPC server who would expose a basic service to queue events.

```protobuf
service MessageQueue {
  rpc SendMessage(...) returns (...) {}
}
```

This would then queue the event in an in-memory priority queue and then send these events to the Event Consumer via a GRPC service exposed by the Event Consumer:

```protobuf
service EventConsumer {
  rpc ProcessMessageFromQueue(...) return (...) {}
}
```

Another option would be to have another component that sits between the queue and the Event Consumer to translate the messages from the various queue formats into a GRPC call to the Event Consumer.

## RabbitMQ

[RabbitMQ](https://www.rabbitmq.com/) is quite similar to Kafka in terms of scalability, however, RabbitMQ is quite easier to set up. It also uses a more generic protocol called [AMQP](https://www.amqp.org/). RabbitMQ supports [priority queues](https://www.rabbitmq.com/priority.html) which makes it a great choice for Certes.

Consuming events from RabbitMQ is quite similar to Kafka since it uses a Go channel.

```go
func main() {
    ...

    // We consume data in the queue named test using the channel we created in go.
    msgs, err := channel.Consume("test", "", false, false, false, false, nil)

    if err != nil {
        panic("error consuming the queue: " + err.Error())
    }

    // We loop through the messages in the queue and print them to the console.
    // The msgs will be a go channel, not an amqp channel
    for msg := range msgs {
        //print the message to the console
        fmt.Println("message received: " + string(msg.Body))
        // Acknowledge that we have received the message so it can be removed from the queue
        msg.Ack(false)
    }

    // We close the connection after the operation has completed.
    defer connection.Close()
}
```

## SQS

The [SQS](https://aws.amazon.com/sqs/) service from AWS is fairly flexible and scalable, there are a few options when it comes to consuming events from the queue. One option is to do long polling and handle any events that were returned (less real-time) or set up a Lambda function with a Cloudwatch event trigger. In an attempt to be platform agnostic, I think we should go for the polling option or add a lambda function as a `contrib` package which could then call the Event Consumer via GRPC.

SQS does not support priority; the solution would be to create N SQS queues for N priorities. This is not ideal as there would need to be some AWS specific set up instructions or implementation details when creating a new priority.

## Kafka

[Kafka](https://kafka.apache.org/) is known for being extremely scalable and handle huge workloads. The downside is that it is pretty difficult to set up, even locally, because of the dependency of Zookeeper. The upside is that it is platform agnostic. Kafka doesn't natively support priorities so we would need to implement a similar method to SQS to create new queues for each new priority.

Consumer events from Kafka is pretty straightforward, under the hood it uses polling but from a client perspective it is just a regular Go channel.

```go
import (
	"fmt"
	"gopkg.in/confluentinc/confluent-kafka-go.v1/kafka"
)

func main() {
	c, err := kafka.NewConsumer(&kafka.ConfigMap{
		"bootstrap.servers": "localhost",
		"group.id":          "myGroup",
		"auto.offset.reset": "earliest",
	})

	if err != nil {
		panic(err)
	}

	c.SubscribeTopics([]string{"myTopic", "^aRegex.*[Tt]opic"}, nil)

	for {
		msg, err := c.ReadMessage(-1)
		if err == nil {
			fmt.Printf("Message on %s: %s\n", msg.TopicPartition, string(msg.Value))
		} else {
			// The client will automatically try to recover from all errors.
			fmt.Printf("Consumer error: %v (%v)\n", err, msg)
		}
	}

	c.Close()
}
```

## What about Google PubSub?

Google [PubSub](https://cloud.google.com/pubsub/docs/overview) seems like it should be on this list, but from my understanding it is too tightly coupled with Google Cloud to be used in a generic way. If I'm wrong, submit a PR :smile:.

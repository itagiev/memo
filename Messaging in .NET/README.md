## Работа с MassTransit

### Message

Message is a unit that cares data to different components (class, record, interface). must be reference type. Should only contain properties
Made of:
- Headers
- Payload
Can be initialized as:
- Anonymous type
- Concrete type

### Delivery modes/guarantees
Delivery modes:
- At most once (fire and forget) - broker deliver message to consumer without any aknowledge from the consumer. Broker will consider message has being delivered. Low latency and high throughput. Messages can be lost
- At least once (aknowledge delivery) - broker ensures that messages are delivered to consumers at least once. Broker waits for an acknowledgment. If consumer aknwoledge message successfully the message will be removed from the queue. If aknowledgment will not recieved in a specified time or time to live of the message the broker will assume that the message was not processed. Can resolved in duplicate message processing if the consumer crushes after processing the message but before sending an aknowledgment
- Exactly once (transactional delivery) - provides strong guarantees of message delivery but may incure higher latency and reduce throughput

### Topologies

The configuration and structure of the message routing

### Endpoints

A messaging address or destination where messages are sent or received

- Receive endpoints - a destination for consuming messages, well known queue where consumer will listen and receive messages for processing, it's tied to specific message queue or topic depending on a transport (rabbitmq, azure service bus...)
- Publish endpoints
- Send endpoints - a destination for sending messages, address where messages are sent explicitly without needing to know about consumers

Other parameters:
- Transport details
- Concurrency and Prefetch limits
- Retry policies

Endpoint abstracts the phisical messaging infrastructure, provides mechanism for sending/receiving or publishing messages in decoupled and transport agnostic way

## Разделы

- [Setup](setup.md)
- [Basics](basics.md)

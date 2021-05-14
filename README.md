# Kafka (Explained)
This repository is for understanding the concept of Kafka and log processing system. The information is referenced mainly by a research paper, Kafka: a Distributed Messaging System for Log Processing.
## What is Kafka?
A distributed messaging system for collecting and delivering high volumes of log data with low latency which initially built for LinkedIn platform.
### Advantages
```
- Strong in ditributed support
- Highly scalable
- High throughout
- API that allows applications to consume log events in real time
- Reliable
```
---
## Log agreegator and messaging system
### Log data
Log data has been a componenet of analytics used to track user engagement, system utilization, and other metrics. The main information that is needed to use for activity data are:
```
- Search relevance 
- Recommendations which may be driven by item popularity or coccurrence in the activity stream
- Ad targeting and reporting
- Security applications 
- Newsfeed features that aggregate user status updates or actions
```
---
## Karka Architecture and Design Principle
### Basic concepts
#### Topics
**Topic** is a particular type of stream of messages
This topic is divided into multiple **partitions** to balance the load.
Then, each broker stores one or more of these partitions. (The concept of brokers is down below.)
#### Brokers
**Borkers** are sets of servers that are storing published messages
#### Publishers
A **publisher** is an entity that publishes the messages to a topic.
#### Consumers
A **consumer** is an entity that subscribes topic from the brokers by pulling the data.
#### Messages
A **message** is a data that is consisited of the content, payload, and additional data.
The Kafka message doesn't have explicit message id but addressed by logical offset in the log. This avoids the overhead. Therefore, the id is computed implicity. The next message id is computed by adding the length of the current message to its id. 
### Sample producer code and consumer code
**Creating a Producer**
```
producer = new Producer(...);
```
**Creating a message**
```
message = new Message("test message str".getBytes());
set = new MessageSet(message);
```
A message is defined to contain just a payload of bytes and a user can choose a serializatio method to encode a message. 
**Producer sends a set of message**
```
producer.send("topic1", set);
```
The producer can send a set of messages in a single publish request.
### Delivery Models
#### (1) Point-to-point delivery model
#### (2) Publish/subscribe model

### Architecture of the storage
####  A producer publishes message to parition.
- A partition of a topic is regarded as a logical log. This logical log is implemented as a set of segment files of approximately same size (think it as pages in paged/segmented virtual memory).
- When a producer publishes a message to a partition, it append a message to the last segmented files. (The system flush the segment files to disk only after messages have been published or certain period of time.)
####  A consumer receives messages and issue asynchrononous pull requests to the broker.
- A consumer always receives messages from a particular partition sequentially. Acknowledging a particular message offset means it has received all messages prior to the offset. (similar to sequence number in stop and wait protocol)
- Pull requests created by a consumer consists of offset of the starting message and a number of bytes to fetch.
- A consumer will recieves a buffer of data read for the application to consume.
- A consumer computes the offset of the next message and uses it in the next pull request. (The concept of offset and id, which is mentioned above, are interchangeable)
- EAch pull request retrieves multiple messages up to a certain size
####  A broker locates the segment file
- Each broker has a sorted list of offsets
- A broker searches the offset list and find the location of the segment file of the requested message and send ti back to the consumer.

### Architecture of the transfer system
- A producer submits a message in a single send request
- Avoid explicitly chacing messages but rather use **underlying file system page cache**. Advantages of it listed below:
```
Avoid double buffering.
Retain a benefit of warm cache.
By having a very little overhead in garbage collecting, it is efficient to implment in a VM-based language.
Small amount, normal operating system caching heurisitcs are very effective.
The performance of production and consumption are both linear time to the data size.
```
- Exploit the sendfile API to efficintly deliver bytes in a log segment file.

### Operates stateless broker.
The information about how much each consumer has consumed is maintained by consumer, not by a broker.

**Challenge of deleting a message**: Kafka solve this problem by using a simple time-based SLA for the retention policy. A message is automatically deleted by certain period of time.

**Benefits**:

- Reduces a lot of complexity and the overhead on the broker.
- A performance doesn't degrade with a larger data.
- A consumer can rewid back to an old offset and re-consume data.

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
##### Consumer groups
A **consumer group** is a set of consumers (either one or more) that jointily consume a set of subscribed topics.
(Cosumers in the same group can be in different processes or on different machines.)
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
---
## Kafka's Distributed Coordination
**Challenge**: Divide the messages evenly among the consumers without too much coordination overhead.
### First decision: All messages from one partition are consumed only by a single consumer within each consumer group.
- Reducing amount of overhead by eliminimating locking and state maintenance overhead.
- The overhead is only needed when consuming process requires co-ordinate process to rebalance the load.
- **Note**: To truely balance the load, we need many more partitions in a topic than the consumers in each group.
### Second deicison: Let comsumers coordinate among themselves without a master node by employing consensus service Zookeeper.
Zookeeper provides following services:
- Register a watcher on a path and get notified when the children of a path or the value of a path has changed.
- A path can be created as ephemeral which automatically remove a path by the Zookeeper server when the client is gone.
- Zookeeper replicates its data to multiple servers.
  
By using Zookeeper Kafka can easily:
- Detect the addition and the removal of brokers and consumers.
- Trigger a rebalance process in each consumer when the addition and removal happens.
- Maintain the consumption relationship and keep track of the consumed offset of each partition. 
  
#### Registries in Zookeeper
When each broker or consumer starts up, it stores its information in a broker or consumer registry in Zookeeper.
##### Broker registry
Contains:
- The broker’s host name
- The broker’s host port 
- A set of topics and partitions that is being stored
 The paths are ephermeral for the broker registery.
##### Consumer registery
Contains:
- The consumer group it belongs to
- Set of topics that it subscribes to
 The paths are ephermeral for the consumer registery.
##### Ownership registery
Each consumer group associated with an ownership register.
Contains:
- A path for each subscribed partition (id of consumer current consuming)
 The paths are ephermeral for the owndership registery.
##### Offset registery
Each consumer group associated with an offset register.
Contains: 
 - An offset of the last consumed message in a partition for each subscribed partition
 The paths are persistent for the offset registery.
---
## Delivery Guarantees
### Failed consumer recovery
When a consumer process crashes without a clean shutdown, it may get duplicates. If an application cares about duplicate, it must add its own deduplication logic.
### Log corruption
To guarantee on the cordering of meesages from a single partiton (avoid confusion by the orderings of other partitions), Kafka stores a CRC for each message in log. CRC allows to check network errors after message is produced or consumed.
### Failed Brokers
If a broker goes down, any message stored on it not yet consumed becomes unavailable. If the storage system on a broker is permanently damaged, any unconsumed message is lost forever

---
## Reference
Kreps, J., Narkhede, N., & Rao, J. (2011, June). Kafka: A distributed messaging system for log processing. In _Proceedings of the NetDB_ (Vol. 11, pp. 1-7).

# Kafka (Explained)
This repository is for understanding the concept of Kafka and log processing system. The information is referenced mainly by a research paper, Kafka: a Distributed Messaging System for Log Processing.
## What is Kafka?
A distributed messaging system for collecting and delivering high volumes of log data with low latency which initially built for LinkedIn platform.
### Advantages
(1) Strong in ditributed support
(2) highly scalable
(3) high throughout
(4) API that allows applications to consume log events in real time
(5) reliable

---
## Log agreegator and messaging system
### Log data
Log data has been a componenet of analytics used to track user engagement, system utilization, and other metrics. The main information that is needed to use for activity data are (1) search relevance, (2) 
recommendations which may be driven by item popularity or coccurrence in the activity stream, (3) ad targeting and reporting, and (4) security applications (5) newsfeed features that aggregate user status updates or actions.

---
## Karka Architecture and Design Principle
### Basic concepts
#### (1) Topics
**Topic** is a particular type of stream of messages
This topic is divided into multiple **partitions** to balance the load.
Then, each broker stores one or more of these partitions. (The concept of brokers is down below.)
#### (2) Brokers
**Borkers** are sets of servers that are storing published messages
#### (3) Publishers
A **publisher** is an entity that publishes the messages to a topic.
#### (4) Consumers
A **consumer** is an entity that subscribes topic from the brokers by pulling the data.
#### (5) Messages
A **message** is a data that is consisited of the content, payload, and additional data.
The kKafka message doesn't have explicit message id but addressed by logical offset in the log. This avoids the overhead. Therefore, the id is computed implicity. The next message id is computed by adding the length of the current message to its id. 
### Delivery Models
#### (1) Point-to-point delivery model
#### (2) Publish/subscribe model
### Architecture of the storage
(1) A partition of a topic is regarded as a logical log. This logical log is implemented as a set of segment files of approximately same size (think it as pages in paged/segmented virtual memory).
(2) When a producer publishes a message to a partition, it append a message to the last segmented files.
(3) Flush the segment files to disk only after messages have been published or certain period of time.

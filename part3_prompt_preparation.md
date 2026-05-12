3.1.1 Repository Context

aiokafka GitHub Repository is an asynchronous Apache Kafka client library built for Python using the asyncio framework. The repository enables Python applications to communicate with Apache Kafka brokers in a fully non blocking manner. It provides asynchronous producer and consumer implementations that allow developers to send, receive, and process streaming data efficiently while maintaining high concurrency and scalability. 

The intended users of the repository are backend developers, infrastructure engineers, and teams building distributed systems or real-time data pipelines. Since many large-scale systems rely on Kafka for reliable communication between services, aiokafka helps Python developers integrate Kafka into asynchronous applications without blocking execution threads.

The repository addresses the domain of distributed event streaming and asynchronous communication. One of the major challenges in this domain is handling high-throughput messaging while maintaining reliability and responsiveness. aiokafka solves problems related to asynchronous task coordination, message fetching, offset management, consumer group synchronization, and fault tolerance. Because Kafka consumers operate continuously in production environments, the repository also focuses heavily on stable task lifecycle management, error handling, and graceful recovery during network interruptions or broker failures.

3.1.2 Pull Request Description

This pull request introduces two new convenience methods to the aiokafka GitHub Repository consumer API: seek\_to\_beginning() and seek\_to\_end(). These methods simplify how developers move a Kafka consumer’s read position within a partition. Before this PR, if a developer wanted to reset the consumer to the oldest available message or skip directly to the newest offset, they had to manually request the offset values from the Kafka broker and then call consumer.seek(partition, offset) themselves. This process required multiple steps and added unnecessary complexity to consumer management.

The new seek\_to\_beginning() method moves the consumer position to the earliest available offset in one or more partitions, allowing the consumer to re-read all existing messages from the start. The seek\_to\_end() method moves the position to the latest offset so that the consumer ignores old messages and only processes new records arriving after the seek operation. Both methods accept optional TopicPartition arguments. If no partitions are provided, the operation automatically applies to all currently assigned partitions.

The PR also introduces input validation and state checks. A TypeError is raised if invalid argument types are passed, while an IllegalStateError is raised if the specified partition is not currently assigned to the consumer. Compared to the previous multi-step workflow, the new implementation provides a cleaner single-method asynchronous API. The behavior now closely matches the official Java Kafka client, improving API consistency across Kafka ecosystems and making the Python client easier for Kafka developers to use.

3.1.3 Acceptance Criteria

✓ When seek\_to\_beginning() is called on an assigned partition, the next getone() call returns the earliest available message in that partition.

✓ When seek\_to\_end() is called, the consumer skips all existing messages and only receives new messages published after the seek operation.

✓ When no partitions are provided, both methods apply automatically to all currently assigned partitions.

✓ When a non-TopicPartition argument is passed, the consumer raises a TypeError.

✓ When an unassigned partition is passed, the consumer raises an IllegalStateError before attempting any broker request.

✓ Both methods correctly support multiple partitions in a single method call.

✓ Calling seek\_to\_beginning() multiple times produces consistent offset reset behavior.

✓ The methods preserve existing consumer functionality without breaking manual seek() operations.

3.1.4 Edge Cases

1. Empty Topic  
   If seek\_to\_beginning() is called on a topic partition with no messages, the consumer offset should remain at 0, and subsequent getone() calls should wait for new messages instead of failing.  
2. Partition Reassignment During Rebalance  
   If a partition becomes unassigned due to a rebalance between the seek operation and the next fetch request, the consumer should raise an appropriate state error and avoid inconsistent offset behavior.  
3. Invalid Argument Types  
   If a user passes objects that are not TopicPartition instances, such as strings or integers, the methods must immediately raise a TypeError.  
4. Calling Methods before Consumer Start  
   If seek\_to\_beginning() or seek\_to\_end() is called before consumer.start() completes, the consumer should raise a clear error instead of silently failing.  
5. Multiple Partition Seeks with Mixed Validity  
   If multiple partitions are passed and one partition is unassigned, the implementation should fail safely without partially applying offset changes.  
   

## 

3.1.5 Initial Prompt

You are working on the aiokafka GitHub Repository, an asynchronous Apache Kafka client for Python built using asyncio. The repository provides the AIOKafkaConsumer class, which allows Python applications to consume Kafka messages asynchronously while supporting partition assignment, offset management, and non-blocking message processing.

Your task is to implement two new asynchronous convenience methods for the AIOKafkaConsumer class:

* seek\_to\_beginning(\*partitions)  
* seek\_to\_end(\*partitions)

The implementation should be added in:

aiokafka/consumer/[consumer.py](http://consumer.py)

### **Required Behavior**

#### seek\_to\_beginning(\*partitions)

This method should move the consumer position to the earliest available offset for the specified partitions. If no partitions are provided, the method should automatically apply to all currently assigned partitions. After calling this method, the next consumed message should be the oldest available message in each selected partition.

#### seek\_to\_end(\*partitions)

This method should move the consumer position to the latest available offset for the specified partitions. If no partitions are provided, it should apply to all currently assigned partitions. After calling this method, the consumer should skip all existing messages and only receive new messages produced after the seek operation.

### **Validation Requirements**

* Validate that every provided argument is an instance of TopicPartition.  
* Raise TypeError if invalid argument types are passed.  
* Raise IllegalStateError if any partition is not currently assigned to the consumer.  
* Ensure validation occurs before any broker communication is attempted.

### **Behavioral Requirements**

* Preserve compatibility with existing manual seek() functionality.  
* Match the behavior of the official Java Kafka client wherever possible.  
* Ensure the methods work correctly with single or multiple partitions.  
* If no partitions are provided, automatically use all assigned partitions.  
* Support asynchronous execution patterns used throughout the repository.

### **Edge Cases to Handle**

* Empty topics with no available messages.  
* Consumer rebalance events occurring during seek operations.  
* Calling the methods before consumer.start().  
* Multiple partition requests containing invalid or unassigned partitions.  
* Concurrent usage alongside ongoing fetch operations.

### **Testing Requirements**

Add or update tests in:

tests/test\_consumer.py

Tests should verify:

* Correct offset positioning after each method call.  
* Proper handling of multiple partitions.  
* Correct exceptions for invalid arguments.  
* Correct exceptions for unassigned partitions.  
* Behavior when topics are empty.  
* Compatibility with existing consumer workflows.  
* Consistent behavior during rebalance scenarios.

Follow the repository’s existing coding style, asynchronous patterns, and error-handling conventions.


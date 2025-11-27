Absolutely! Let me explain **Apache Kafka** step by step with Java examples.

## What is Apache Kafka?

**Apache Kafka** is a distributed streaming platform and message broker. Think of it as a high-performance messaging system that can handle millions of messages per second.

### Simple Analogy

Imagine a **newspaper delivery system**:

- **Producers** = Journalists writing articles
- **Kafka Topics** = Different newspaper sections (Sports, News, Entertainment)
- **Consumers** = Readers subscribing to specific sections
- **Brokers** = Distribution centers
- **Partitions** = Multiple delivery routes for same section

---

## Why Use Kafka?

**Traditional Direct Communication:**

```
Service A → Service B
Service A → Service C
Service A → Service D
```

- Tight coupling ❌
- Service A must know all consumers ❌
- If Service B is down, message lost ❌

**With Kafka:**

```
Service A → Kafka Topic ← Service B
                        ← Service C
                        ← Service D
```

- Loose coupling ✅
- Service A doesn't know consumers ✅
- Messages stored, can be replayed ✅
- Horizontal scalability ✅

---

## Core Concepts

### 1. **Topic**

- A category or feed name
- Like a database table or a folder
- Messages are published to topics

### 2. **Producer**

- Applications that publish (write) messages to topics
- Like INSERT in databases

### 3. **Consumer**

- Applications that subscribe to (read) messages from topics
- Like SELECT in databases

### 4. **Broker**

- Kafka server that stores data and serves clients
- A Kafka cluster has multiple brokers

### 5. **Partition**

- Topics are split into partitions for parallelism
- Each partition is an ordered, immutable sequence of messages
- Messages in a partition have a sequential ID called **offset**

### 6. **Consumer Group**

- Multiple consumers working together
- Each message consumed by only one consumer in the group
- Enables parallel processing

---

## The Architecture

```
Producer 1 ─┐
Producer 2 ─┼──→ Topic: "orders" ──→ Partition 0 ──→ Consumer Group A
Producer 3 ─┘         ↓                  ↓              ↓
                      └──→ Partition 1 ──┘              Consumer 1
                      └──→ Partition 2 ─────────────→ Consumer 2
```

---

## Step-by-Step Setup and Example

---

## Step 1: Installation and Setup

### Option A: Docker (Easiest)

Create `docker-compose.yml`:

```yaml
version: "3"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

Start Kafka:

```bash
docker-compose up -d
```

### Option B: Local Installation

1. Download from https://kafka.apache.org/downloads
2. Extract and navigate to directory
3. Start Zookeeper:
   ```bash
   bin/zookeeper-server-start.sh config/zookeeper.properties
   ```
4. Start Kafka:
   ```bash
   bin/kafka-server-start.sh config/server.properties
   ```

---

## Step 2: Add Maven Dependencies

```xml
<dependencies>
    <!-- Kafka Client -->
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>3.6.0</version>
    </dependency>

    <!-- SLF4J for logging (optional but recommended) -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

---

## Step 3: Create a Simple Producer

### Basic Producer Example

```java
package com.example.kafka;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class SimpleProducer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        System.out.println("=== Starting Kafka Producer ===\n");

        // STEP 1: Configure Producer
        Properties props = new Properties();

        // Kafka broker address
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        // Serializers - convert objects to bytes
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                  StringSerializer.class.getName());

        // Optional: Client ID for identification
        props.put(ProducerConfig.CLIENT_ID_CONFIG, "my-producer");


        // STEP 2: Create Producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);


        // STEP 3: Send Messages
        String topic = "my-topic";

        for (int i = 1; i <= 10; i++) {

            String key = "key-" + i;
            String value = "Message number " + i;

            // Create a record (message)
            ProducerRecord<String, String> record =
                new ProducerRecord<>(topic, key, value);

            // Send the record
            System.out.println("Sending: " + key + " = " + value);

            // Synchronous send (waits for acknowledgment)
            RecordMetadata metadata = producer.send(record).get();

            System.out.println("Sent to partition " + metadata.partition() +
                             " with offset " + metadata.offset());

            Thread.sleep(500); // Slow down for demo
        }


        // STEP 4: Close the producer
        System.out.println("\n=== Closing Producer ===");
        producer.close();
    }
}
```

---

## Step-by-Step Producer Breakdown

### Step 1: Configure Producer

```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
```

**What this does:**

- `BOOTSTRAP_SERVERS_CONFIG`: Kafka broker addresses (comma-separated if multiple)
- This is how the producer finds the Kafka cluster
- The producer will discover all other brokers automatically

### Key Configuration Properties:

```java
// REQUIRED
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

// OPTIONAL (common ones)
props.put(ProducerConfig.ACKS_CONFIG, "all");  // Wait for all replicas
props.put(ProducerConfig.RETRIES_CONFIG, 3);   // Retry on failure
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);  // Batch messages
props.put(ProducerConfig.LINGER_MS_CONFIG, 1);  // Wait before sending batch
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");  // Compress data
```

### Step 2: Create a Record

```java
ProducerRecord<String, String> record =
    new ProducerRecord<>(topic, key, value);
```

**ProducerRecord components:**

- **Topic**: Where to send the message
- **Key**: Optional, used for partitioning (messages with same key go to same partition)
- **Value**: The actual message content
- **Partition**: Optional, manual partition selection
- **Timestamp**: Optional, defaults to current time

### Step 3: Send the Message

```java
// Fire and forget (fastest, no guarantees)
producer.send(record);

// Asynchronous with callback
producer.send(record, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception == null) {
            System.out.println("Success! Offset: " + metadata.offset());
        } else {
            System.err.println("Error: " + exception.getMessage());
        }
    }
});

// Synchronous (slowest, strongest guarantees)
RecordMetadata metadata = producer.send(record).get();
```

---

## Step 4: Create a Simple Consumer

### Basic Consumer Example

```java
package com.example.kafka;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class SimpleConsumer {

    public static void main(String[] args) {

        System.out.println("=== Starting Kafka Consumer ===\n");

        // STEP 1: Configure Consumer
        Properties props = new Properties();

        // Kafka broker address
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        // Consumer group - consumers in same group share the work
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group");

        // Deserializers - convert bytes to objects
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                  StringDeserializer.class.getName());

        // Where to start reading if no offset exists
        // "earliest" = from beginning, "latest" = only new messages
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // Auto-commit offsets (or handle manually)
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");


        // STEP 2: Create Consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);


        // STEP 3: Subscribe to topic(s)
        String topic = "my-topic";
        consumer.subscribe(Collections.singletonList(topic));

        System.out.println("Subscribed to topic: " + topic);
        System.out.println("Waiting for messages...\n");


        // STEP 4: Poll for messages (continuous loop)
        try {
            while (true) {

                // Poll for new messages (wait up to 100ms)
                ConsumerRecords<String, String> records =
                    consumer.poll(Duration.ofMillis(100));

                // Process each message
                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("─────────────────────────────────");
                    System.out.println("Topic: " + record.topic());
                    System.out.println("Partition: " + record.partition());
                    System.out.println("Offset: " + record.offset());
                    System.out.println("Key: " + record.key());
                    System.out.println("Value: " + record.value());
                    System.out.println("Timestamp: " + record.timestamp());

                    // Process the message here
                    processMessage(record.value());
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // STEP 5: Close the consumer
            System.out.println("\n=== Closing Consumer ===");
            consumer.close();
        }
    }

    private static void processMessage(String message) {
        // Your business logic here
        System.out.println("Processing: " + message);
    }
}
```

---

## Step-by-Step Consumer Breakdown

### Step 1: Consumer Group

```java
props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group");
```

**What is a Consumer Group?**

- Multiple consumers working together as a team
- Each partition assigned to only one consumer in the group
- Enables parallel processing
- If one consumer dies, its partitions are reassigned

**Example:**

```
Topic: orders (3 partitions)
Consumer Group: order-processors

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

### Step 2: Offset Management

```java
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
```

**Offset** = Position in the partition

**Options:**

- `earliest`: Start from beginning (replay all messages)
- `latest`: Start from end (only new messages)
- Manual: You control exactly where to start

### Step 3: Polling

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
```

**What happens:**

1. Consumer asks Kafka: "Any new messages?"
2. Kafka returns batch of records (up to `max.poll.records`)
3. Consumer processes the batch
4. Consumer commits offset (marks messages as read)
5. Repeat

**Important:** Must call `poll()` regularly or consumer is considered dead!

---

## Step 5: Complete Example - Order Processing System

Let's build a realistic example: an e-commerce order processing system.

### Order Class

```java
package com.example.kafka.model;

public class Order {
    private String orderId;
    private String customerId;
    private String product;
    private int quantity;
    private double price;

    public Order() {}

    public Order(String orderId, String customerId, String product,
                 int quantity, double price) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.product = product;
        this.quantity = quantity;
        this.price = price;
    }

    // Getters and setters
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }

    public String getCustomerId() { return customerId; }
    public void setCustomerId(String customerId) { this.customerId = customerId; }

    public String getProduct() { return product; }
    public void setProduct(String product) { this.product = product; }

    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }

    public double getTotalPrice() {
        return quantity * price;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderId='" + orderId + '\'' +
                ", customerId='" + customerId + '\'' +
                ", product='" + product + '\'' +
                ", quantity=" + quantity +
                ", price=" + price +
                ", total=" + getTotalPrice() +
                '}';
    }
}
```

### JSON Serializer for Order

```java
package com.example.kafka.serializer;

import com.example.kafka.model.Order;
import com.google.gson.Gson;
import org.apache.kafka.common.serialization.Serializer;

public class OrderSerializer implements Serializer<Order> {

    private Gson gson = new Gson();

    @Override
    public byte[] serialize(String topic, Order order) {
        if (order == null) {
            return null;
        }
        return gson.toJson(order).getBytes();
    }
}
```

### JSON Deserializer for Order

```java
package com.example.kafka.serializer;

import com.example.kafka.model.Order;
import com.google.gson.Gson;
import org.apache.kafka.common.serialization.Deserializer;

public class OrderDeserializer implements Deserializer<Order> {

    private Gson gson = new Gson();

    @Override
    public Order deserialize(String topic, byte[] data) {
        if (data == null) {
            return null;
        }
        return gson.fromJson(new String(data), Order.class);
    }
}
```

### Order Producer

```java
package com.example.kafka;

import com.example.kafka.model.Order;
import com.example.kafka.serializer.OrderSerializer;
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.Random;

public class OrderProducer {

    private static final String TOPIC = "orders";
    private static final String[] PRODUCTS = {"Laptop", "Phone", "Tablet", "Monitor", "Keyboard"};
    private static final Random random = new Random();

    public static void main(String[] args) throws InterruptedException {

        System.out.println("=== Starting Order Producer ===\n");

        // Configure producer
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                  OrderSerializer.class.getName());
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        props.put(ProducerConfig.RETRIES_CONFIG, 3);

        // Create producer
        KafkaProducer<String, Order> producer = new KafkaProducer<>(props);

        // Generate and send orders
        for (int i = 1; i <= 20; i++) {

            // Create a random order
            Order order = new Order(
                "ORD-" + i,
                "CUST-" + random.nextInt(100),
                PRODUCTS[random.nextInt(PRODUCTS.length)],
                random.nextInt(5) + 1,
                random.nextDouble() * 1000 + 100
            );

            // Key = customer ID (ensures all orders from same customer go to same partition)
            String key = order.getCustomerId();

            ProducerRecord<String, Order> record =
                new ProducerRecord<>(TOPIC, key, order);

            // Send with callback
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("✓ Sent order " + order.getOrderId() +
                                         " to partition " + metadata.partition() +
                                         " | Total: $" +
                                         String.format("%.2f", order.getTotalPrice()));
                    } else {
                        System.err.println("✗ Error sending order: " + exception.getMessage());
                    }
                }
            });

            Thread.sleep(500); // Simulate orders coming in
        }

        // Wait for all messages to be sent
        producer.flush();
        producer.close();

        System.out.println("\n=== All orders sent ===");
    }
}
```

### Order Consumer (Inventory Service)

```java
package com.example.kafka;

import com.example.kafka.model.Order;
import com.example.kafka.serializer.OrderDeserializer;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class InventoryService {

    private static final String TOPIC = "orders";
    private static final String GROUP_ID = "inventory-service";

    public static void main(String[] args) {

        System.out.println("=== Starting Inventory Service ===\n");

        // Configure consumer
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                  OrderDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false"); // Manual commit

        // Create consumer
        KafkaConsumer<String, Order> consumer = new KafkaConsumer<>(props);

        // Subscribe
        consumer.subscribe(Collections.singletonList(TOPIC));

        System.out.println("Listening for orders...\n");

        try {
            while (true) {
                ConsumerRecords<String, Order> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, Order> record : records) {
                    Order order = record.value();

                    System.out.println("─────────────────────────────────");
                    System.out.println("[INVENTORY] Processing order: " + order.getOrderId());
                    System.out.println("  Product: " + order.getProduct());
                    System.out.println("  Quantity: " + order.getQuantity());

                    // Simulate inventory check
                    if (checkInventory(order)) {
                        System.out.println("  ✓ Inventory available");
                        reserveInventory(order);
                    } else {
                        System.out.println("  ✗ Insufficient inventory");
                    }
                }

                // Manually commit offsets after processing batch
                consumer.commitAsync();
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    private static boolean checkInventory(Order order) {
        // Simulate inventory check
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return true; // Simplified
    }

    private static void reserveInventory(Order order) {
        // Simulate reserving inventory
        System.out.println("  → Reserved " + order.getQuantity() +
                         " units of " + order.getProduct());
    }
}
```

### Order Consumer (Payment Service)

```java
package com.example.kafka;

import com.example.kafka.model.Order;
import com.example.kafka.serializer.OrderDeserializer;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class PaymentService {

    private static final String TOPIC = "orders";
    private static final String GROUP_ID = "payment-service";

    public static void main(String[] args) {

        System.out.println("=== Starting Payment Service ===\n");

        // Configure consumer
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                  OrderDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // Create consumer
        KafkaConsumer<String, Order> consumer = new KafkaConsumer<>(props);

        // Subscribe
        consumer.subscribe(Collections.singletonList(TOPIC));

        System.out.println("Listening for orders...\n");

        try {
            while (true) {
                ConsumerRecords<String, Order> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, Order> record : records) {
                    Order order = record.value();

                    System.out.println("─────────────────────────────────");
                    System.out.println("[PAYMENT] Processing order: " + order.getOrderId());
                    System.out.println("  Customer: " + order.getCustomerId());
                    System.out.println("  Total: $" + String.format("%.2f", order.getTotalPrice()));

                    // Simulate payment processing
                    if (processPayment(order)) {
                        System.out.println("  ✓ Payment successful");
                    } else {
                        System.out.println("  ✗ Payment failed");
                    }
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    private static boolean processPayment(Order order) {
        // Simulate payment processing
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("  → Charged $" +
                         String.format("%.2f", order.getTotalPrice()) +
                         " to customer " + order.getCustomerId());
        return true;
    }
}
```

---

## Running the Complete Example

### Terminal 1: Start Producer

```bash
java com.example.kafka.OrderProducer
```

### Terminal 2: Start Inventory Service

```bash
java com.example.kafka.InventoryService
```

### Terminal 3: Start Payment Service

```bash
java com.example.kafka.PaymentService
```

**What happens:**

1. Producer sends 20 orders to "orders" topic
2. Each order goes to a partition (based on customer ID key)
3. Both services consume ALL messages (different consumer groups)
4. Each service processes orders independently
5. Services can be scaled independently

---

## Step 6: Understanding Partitions

### How Partitioning Works

```java
// With key - same key always goes to same partition
ProducerRecord<String, Order> record =
    new ProducerRecord<>(topic, "CUST-123", order);

// Without key - round-robin distribution
ProducerRecord<String, Order> record =
    new ProducerRecord<>(topic, order);

// Manual partition selection
ProducerRecord<String, Order> record =
    new ProducerRecord<>(topic, 2, "CUST-123", order);
```

### Partition Assignment in Consumer Group

```
Topic: orders (3 partitions)
Consumer Group: processors (2 consumers)

Consumer 1 → Partition 0, Partition 1
Consumer 2 → Partition 2

If Consumer 3 joins:
Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2

If Consumer 1 dies:
Consumer 2 → Partition 0, Partition 1
Consumer 3 → Partition 2
```

---

## Step 7: Kafka with Spring Boot

For production applications, use Spring Kafka:

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

### application.yml

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: my-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: com.example.kafka.model
```

### Spring Producer

```java
@Service
public class OrderService {

    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;

    public void createOrder(Order order) {
        kafkaTemplate.send("orders", order.getCustomerId(), order);
    }
}
```

### Spring Consumer

```java
@Service
public class OrderListener {

    @KafkaListener(topics = "orders", groupId = "inventory-service")
    public void listen(Order order) {
        System.out.println("Received order: " + order);
        // Process order
    }
}
```

Much simpler!

---

## Step 8: Important Kafka Concepts

### 1. **Replication**

```
Topic: orders (replication-factor: 3)

Partition 0: Broker 1 (Leader), Broker 2 (Follower), Broker 3 (Follower)
Partition 1: Broker 2 (Leader), Broker 1 (Follower), Broker 3 (Follower)
```

- If Broker 1 fails, Broker 2 becomes leader for Partition 0
- No data loss!

### 2. **Offset Management**

```
Partition 0:
[Msg-0] [Msg-1] [Msg-2] [Msg-3] [Msg-4] [Msg-5]
   ↑                           ↑
 Offset 0                  Consumer at offset 4
```

- Offset tracks which messages have been read
- Committed offset = last successfully processed message
- Can replay by resetting offset

### 3. **Message Ordering**

- ✅ Guaranteed within a partition
- ❌ Not guaranteed across partitions
- Use keys to ensure related messages go to same partition

### 4. **Retention**

```java
// Messages kept for 7 days (default)
log.retention.hours=168

// Or by size
log.retention.bytes=1073741824  // 1GB
```

---

## Common Use Cases

### 1. **Event Sourcing**

Store all changes as events

```
orders topic:
- OrderCreated
- OrderPaid
- OrderShipped
- OrderDelivered
```

### 2. **Log Aggregation**

Collect logs from multiple services

```
Service A → logs topic
Service B → logs topic
Service C → logs topic
           ↓
      Log Analyzer
```

### 3. **Stream Processing**

Real-time data processing

```
Raw Data → Kafka → Stream Processor → Processed Data → Kafka
```

### 4. **Metrics Collection**

```
App Metrics → Kafka → Monitoring System
```

### 5. \*\*Microservices

Communication\*\*

```
Order Service → Kafka → Inventory Service
                      → Payment Service
                      → Notification Service
```

---

## Kafka vs Other Messaging Systems

| Feature               | Kafka                    | RabbitMQ       | ActiveMQ             |
| --------------------- | ------------------------ | -------------- | -------------------- |
| **Throughput**        | Very High (millions/sec) | Medium         | Medium               |
| **Message Ordering**  | Per partition            | Per queue      | Per queue            |
| **Message Retention** | Days/weeks               | Until consumed | Until consumed       |
| **Replay**            | Yes ✅                   | No ❌          | No ❌                |
| **Durability**        | Excellent                | Good           | Good                 |
| **Complexity**        | Higher                   | Lower          | Medium               |
| **Use Case**          | High-volume streaming    | Task queues    | Enterprise messaging |

---

## Best Practices

### ✅ Do's

1. Use keys for related messages (ensures ordering)
2. Set appropriate replication factor (3 for production)
3. Monitor consumer lag
4. Use consumer groups for scalability
5. Handle failures gracefully (retries, dead letter queues)
6. Batch messages when possible
7. Use compression (snappy, gzip)

### ❌ Don'ts

1. Don't use Kafka for request-response patterns (use REST/gRPC)
2. Don't store large files (> 1MB) directly in Kafka
3. Don't create too many partitions (overhead)
4. Don't ignore consumer lag warnings
5. Don't change partition count after production (breaks key ordering)

---

## Monitoring Kafka

### Key Metrics to Watch

```
- Under-replicated partitions (should be 0)
- Consumer lag (how far behind consumers are)
- Broker CPU/memory/disk usage
- Network throughput
- Request latency
```

### Tools

- **Kafka Manager** (Yahoo)
- **Confluent Control Center**
- **Burrow** (LinkedIn's consumer lag monitor)
- **Prometheus + Grafana**

---

## Summary

**Kafka workflow:**

1. Producer sends messages to topics
2. Topics divided into partitions (for parallelism)
3. Messages stored on brokers (with replication)
4. Consumers read from partitions
5. Consumer groups enable parallel processing
6. Offsets track progress

**Key benefits:**

- High throughput (millions of messages/second)
- Horizontal scalability (add more brokers/partitions)
- Fault tolerance (replication)
- Message replay (retained for days/weeks)
- Loose coupling between services

**When to use Kafka:**

- ✅ High-volume event streaming
- ✅ Microservices communication
- ✅ Log aggregation
- ✅ Real-time analytics
- ✅ Event sourcing

**When NOT to use Kafka:**

- ❌ Simple request-response (use REST/gRPC)
- ❌ Low message volume (< 1000/day)
- ❌ Need complex routing (use RabbitMQ)
- ❌ Single consumer only

That's Apache Kafka! It's the backbone of modern event-driven architectures and powers systems at LinkedIn, Netflix, Uber, and thousands of other companies.

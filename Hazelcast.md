# Hazelcast - Complete Guide

**Hazelcast** is an **in-memory data grid (IMDG)** - a distributed caching and computing platform. Think of it as a way to store data in memory (RAM) across multiple servers/nodes, making data access extremely fast.

## Why Use Hazelcast?

- **Speed:** Data in RAM is 1000x faster than disk
- **Distributed:** Data spread across multiple servers
- **Scalability:** Add more servers to handle more data
- **High Availability:** If one server fails, data is still available on other servers
- **Caching:** Reduce database load by caching frequently accessed data

## Basic Concepts

Before diving into code, understand these key concepts:

### 1. Cluster

- Multiple Hazelcast instances (nodes) working together
- They automatically discover each other and form a cluster

### 2. Data Structures

Hazelcast provides distributed versions of common Java collections:

- **IMap** - distributed HashMap
- **IQueue** - distributed Queue
- **IList** - distributed List
- **ISet** - distributed Set
- **ITopic** - publish-subscribe messaging

### 3. Partitions

- Data is automatically divided into partitions (default: 271 partitions)
- Each partition is stored on a primary node + backup nodes
- This provides both distribution and redundancy

---

## Step-by-Step Example

Let's build a simple distributed cache for a user management system.

### Step 1: Add Hazelcast Dependency

#### Maven

```xml
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>5.3.6</version>
</dependency>
```

#### Gradle

```gradle
implementation 'com.hazelcast:hazelcast:5.3.6'
```

### Step 2: The User Class

First, we need a simple class to represent our data:

```java
package com.example.model;

import java.io.Serializable;

// MUST implement Serializable for Hazelcast to distribute it
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long id;
    private String username;
    private String email;
    private int age;

    // Constructors
    public User() {}

    public User(Long id, String username, String email, int age) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.age = age;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() {
        return "User{id=" + id + ", username='" + username +
               "', email='" + email + "', age=" + age + "}";
    }
}
```

**Important:** The class **must implement Serializable** because Hazelcast needs to send objects across the network to other nodes.

### Step 3: Basic Hazelcast Instance

Let's create our first Hazelcast instance:

```java
package com.example;

import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.map.IMap;
import com.example.model.User;

public class HazelcastBasicExample {

    public static void main(String[] args) {

        // STEP 1: Create a Hazelcast instance
        // This automatically starts a Hazelcast node
        HazelcastInstance hazelcastInstance = Hazelcast.newHazelcastInstance();

        System.out.println("Hazelcast instance started!");
        System.out.println("Cluster name: " + hazelcastInstance.getCluster().getClusterName());
        System.out.println("Cluster size: " + hazelcastInstance.getCluster().getMembers().size());

        // STEP 2: Get a distributed map
        // This is like a HashMap but distributed across the cluster
        IMap<Long, User> usersMap = hazelcastInstance.getMap("users");

        System.out.println("\n=== Adding users to distributed map ===");

        // STEP 3: Put data into the map
        usersMap.put(1L, new User(1L, "john_doe", "john@example.com", 25));
        usersMap.put(2L, new User(2L, "jane_smith", "jane@example.com", 30));
        usersMap.put(3L, new User(3L, "bob_jones", "bob@example.com", 35));

        System.out.println("Added " + usersMap.size() + " users");

        // STEP 4: Retrieve data from the map
        System.out.println("\n=== Retrieving users ===");
        User user1 = usersMap.get(1L);
        System.out.println("Retrieved: " + user1);

        // STEP 5: Check if key exists
        System.out.println("\n=== Checking existence ===");
        System.out.println("User with ID 1 exists? " + usersMap.containsKey(1L));
        System.out.println("User with ID 99 exists? " + usersMap.containsKey(99L));

        // STEP 6: Iterate through all entries
        System.out.println("\n=== All users in cache ===");
        for (User user : usersMap.values()) {
            System.out.println(user);
        }

        // STEP 7: Update a user
        System.out.println("\n=== Updating user ===");
        User userToUpdate = usersMap.get(2L);
        userToUpdate.setEmail("jane.new@example.com");
        usersMap.put(2L, userToUpdate);
        System.out.println("Updated: " + usersMap.get(2L));

        // STEP 8: Remove a user
        System.out.println("\n=== Removing user ===");
        usersMap.remove(3L);
        System.out.println("Remaining users: " + usersMap.size());

        // STEP 9: Shutdown
        System.out.println("\n=== Shutting down ===");
        hazelcastInstance.shutdown();
    }
}
```

### Step-by-Step Breakdown

##### Step 1: Create Hazelcast Instance

```java
HazelcastInstance hazelcastInstance = Hazelcast.newHazelcastInstance();
```

**What happens:**

- Hazelcast starts a new node
- It looks for other Hazelcast nodes on the network
- If found, it joins their cluster
- If not found, it creates a new cluster
- Uses default configuration (multicast discovery on local network)

##### Step 2: Get a Distributed Map

```java
IMap<Long, User> usersMap = hazelcastInstance.getMap("users");
```

**What this does:**

- Gets (or creates) a distributed map named "users"
- The map is shared across all nodes in the cluster
- If another node asks for `getMap("users")`, they get the **same map**
- `IMap` is like Java's `Map` interface but distributed

#### Step 3: Put Data

```java
usersMap.put(1L, new User(1L, "john_doe", "john@example.com", 25));
```

**What happens behind the scenes:**

1. Hazelcast calculates which partition this key belongs to
2. Hazelcast determines which node owns that partition
3. If it's this node, data is stored locally
4. If it's another node, data is sent to that node
5. A backup copy is stored on another node for redundancy
6. All this happens automatically and transparently

### Step 4: Retrieve Data

```java
User user1 = usersMap.get(1L);
```

**What happens:**

1. Hazelcast calculates which partition owns this key
2. Hazelcast finds which node has that partition
3. If it's this node, retrieve locally (super fast!)
4. If it's another node, fetch from that node over the network
5. Return the User object

---

## Step 4: Running Multiple Nodes (The Real Power)

Let's see Hazelcast's distributed nature by running multiple nodes:

### Node 1 - First Application

```java
package com.example;

import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.map.IMap;
import com.example.model.User;

public class Node1Application {

    public static void main(String[] args) throws InterruptedException {

        System.out.println("===== STARTING NODE 1 =====");

        // Create first node
        HazelcastInstance hz = Hazelcast.newHazelcastInstance();

        System.out.println("Node 1 started");
        System.out.println("Cluster size: " + hz.getCluster().getMembers().size());

        // Get the distributed map
        IMap<Long, User> usersMap = hz.getMap("users");

        // Add some users from Node 1
        System.out.println("\n[Node 1] Adding users...");
        usersMap.put(1L, new User(1L, "alice", "alice@example.com", 28));
        usersMap.put(2L, new User(2L, "bob", "bob@example.com", 32));

        System.out.println("[Node 1] Added 2 users. Total in cache: " + usersMap.size());

        // Keep the node running
        System.out.println("\n[Node 1] Monitoring the map...");
        while (true) {
            Thread.sleep(5000); // Check every 5 seconds
            System.out.println("[Node 1] Current cache size: " + usersMap.size());

            // Print all users
            System.out.println("[Node 1] Users in cache:");
            for (User user : usersMap.values()) {
                System.out.println("  - " + user);
            }
        }
    }
}
```

### Node 2 - Second Application

```java
package com.example;

import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.map.IMap;
import com.example.model.User;

public class Node2Application {

    public static void main(String[] args) throws InterruptedException {

        // Wait a bit for Node 1 to start
        Thread.sleep(2000);

        System.out.println("===== STARTING NODE 2 =====");

        // Create second node - it will automatically join Node 1's cluster
        HazelcastInstance hz = Hazelcast.newHazelcastInstance();

        System.out.println("Node 2 started");
        System.out.println("Cluster size: " + hz.getCluster().getMembers().size());

        // Get the SAME distributed map as Node 1
        IMap<Long, User> usersMap = hz.getMap("users");

        // We should see the users that Node 1 added!
        System.out.println("\n[Node 2] Checking existing users...");
        System.out.println("[Node 2] Cache size: " + usersMap.size());

        for (User user : usersMap.values()) {
            System.out.println("[Node 2] Found user from Node 1: " + user);
        }

        // Add more users from Node 2
        System.out.println("\n[Node 2] Adding more users...");
        usersMap.put(3L, new User(3L, "charlie", "charlie@example.com", 25));
        usersMap.put(4L, new User(4L, "diana", "diana@example.com", 29));

        System.out.println("[Node 2] Added 2 more users. Total: " + usersMap.size());

        // Retrieve a user added by Node 1
        System.out.println("\n[Node 2] Getting user ID=1 (added by Node 1)...");
        User alice = usersMap.get(1L);
        System.out.println("[Node 2] Retrieved: " + alice);

        // Keep running
        while (true) {
            Thread.sleep(5000);
            System.out.println("[Node 2] Current cache size: " + usersMap.size());
        }
    }
}
```

### What Happens When You Run Both:

1. **Start Node 1:**

   - Creates a cluster
   - Adds 2 users (alice, bob)
   - Cluster size: 1

2. **Start Node 2 (in a separate terminal/IDE):**

   - Discovers Node 1
   - Joins the cluster
   - **Cluster size: 2**
   - Immediately sees the 2 users Node 1 added!
   - Adds 2 more users (charlie, diana)

3. **Node 1 automatically sees:**
   - The 2 new users Node 2 added
   - Total users: 4

**This is the magic of Hazelcast** - data is automatically shared across all nodes!

---

## Step 5: Configuration

You can customize Hazelcast with a configuration file:

### hazelcast.xml (or hazelcast.yaml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hazelcast xmlns="http://www.hazelcast.com/schema/config"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.hazelcast.com/schema/config
           http://www.hazelcast.com/schema/config/hazelcast-config-5.3.xsd">

    <!-- Cluster name - nodes must have same name to join -->
    <cluster-name>my-cluster</cluster-name>

    <!-- Network configuration -->
    <network>
        <port auto-increment="true">5701</port>

        <join>
            <!-- Multicast discovery (default, works on local network) -->
            <multicast enabled="true">
                <multicast-group>224.2.2.3</multicast-group>
                <multicast-port>54327</multicast-port>
            </multicast>

            <!-- TCP/IP discovery (for specific IPs) -->
            <tcp-ip enabled="false">
                <member>192.168.1.100</member>
                <member>192.168.1.101</member>
            </tcp-ip>
        </join>
    </network>

    <!-- Map configuration -->
    <map name="users">
        <!-- Time to live (TTL) - remove entries after 1 hour -->
        <time-to-live-seconds>3600</time-to-live-seconds>

        <!-- Max idle time - remove if not accessed for 30 minutes -->
        <max-idle-seconds>1800</max-idle-seconds>

        <!-- Eviction policy when map is full -->
        <eviction eviction-policy="LRU" max-size-policy="PER_NODE" size="10000"/>

        <!-- Number of backup copies -->
        <backup-count>1</backup-count>

        <!-- Async backup count -->
        <async-backup-count>0</async-backup-count>
    </map>

</hazelcast>
```

### Using the Configuration

```java
import com.hazelcast.config.Config;
import com.hazelcast.config.XmlConfigBuilder;

// Load from XML file
Config config = new XmlConfigBuilder("hazelcast.xml").build();
HazelcastInstance hz = Hazelcast.newHazelcastInstance(config);

// Or programmatic configuration
Config config = new Config();
config.setClusterName("my-cluster");
config.getNetworkConfig().setPort(5701);
HazelcastInstance hz = Hazelcast.newHazelcastInstance(config);
```

---

## Step 6: Common Use Cases

### Use Case 1: Caching Database Queries

```java
public class UserService {
    private HazelcastInstance hazelcast;
    private IMap<Long, User> userCache;
    private UserRepository userRepository; // Database access

    public UserService() {
        this.hazelcast = Hazelcast.newHazelcastInstance();
        this.userCache = hazelcast.getMap("users");
    }

    public User getUserById(Long id) {
        // Try cache first
        User user = userCache.get(id);

        if (user != null) {
            System.out.println("Cache HIT for user " + id);
            return user;
        }

        // Cache miss - fetch from database
        System.out.println("Cache MISS for user " + id + " - fetching from DB");
        user = userRepository.findById(id);

        if (user != null) {
            // Store in cache for next time
            userCache.put(id, user);
        }

        return user;
    }

    public void updateUser(User user) {
        // Update database
        userRepository.save(user);

        // Update cache
        userCache.put(user.getId(), user);
    }

    public void deleteUser(Long id) {
        // Delete from database
        userRepository.delete(id);

        // Remove from cache
        userCache.remove(id);
    }
}
```

### Use Case 2: Distributed Queue for Task Processing

```java
import com.hazelcast.collection.IQueue;

public class TaskProcessor {

    public static void main(String[] args) throws InterruptedException {
        HazelcastInstance hz = Hazelcast.newHazelcastInstance();

        // Get a distributed queue
        IQueue<String> taskQueue = hz.getQueue("tasks");

        // Producer - add tasks to queue
        new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try {
                    taskQueue.put("Task-" + i);
                    System.out.println("Added: Task-" + i);
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        // Consumer - process tasks from queue
        new Thread(() -> {
            while (true) {
                try {
                    String task = taskQueue.take(); // Blocks until item available
                    System.out.println("Processing: " + task);
                    Thread.sleep(2000); // Simulate work
                    System.out.println("Completed: " + task);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

### Use Case 3: Pub/Sub Messaging

```java
import com.hazelcast.topic.ITopic;
import com.hazelcast.topic.Message;
import com.hazelcast.topic.MessageListener;

public class PubSubExample {

    public static void main(String[] args) throws InterruptedException {
        HazelcastInstance hz = Hazelcast.newHazelcastInstance();

        // Get a topic (like a message channel)
        ITopic<String> topic = hz.getTopic("notifications");

        // Subscriber 1
        topic.addMessageListener(new MessageListener<String>() {
            @Override
            public void onMessage(Message<String> message) {
                System.out.println("[Subscriber 1] Received: " + message.getMessageObject());
            }
        });

        // Subscriber 2
        topic.addMessageListener(new MessageListener<String>() {
            @Override
            public void onMessage(Message<String> message) {
                System.out.println("[Subscriber 2] Received: " + message.getMessageObject());
            }
        });

        // Publisher
        Thread.sleep(1000);
        topic.publish("Hello from Hazelcast!");
        topic.publish("Another message!");

        Thread.sleep(2000);
        hz.shutdown();
    }
}
```

---

## Key Concepts Summary

### 1. **Automatic Distribution**

- Data is automatically spread across nodes
- You don't manage which node stores what

### 2. **Automatic Discovery**

- Nodes find each other automatically (multicast/TCP-IP)
- New nodes join the cluster seamlessly

### 3. **Replication & Backup**

- Each data partition has backup copies
- If a node fails, backups take over

### 4. **Near Cache** (Advanced)

- Keep frequently accessed data in local memory
- Even faster than distributed access

### 5. **Partitioning**

- Data split into 271 partitions by default
- Evenly distributed across nodes

---

## The Flow Diagram

```
Application calls: usersMap.put(1L, user)
         ↓
Hazelcast calculates: hash(1L) → Partition #42
         ↓
Hazelcast finds: Node 2 owns Partition #42
         ↓
Hazelcast sends data to Node 2
         ↓
Node 2 stores the data locally
         ↓
Node 2 sends backup to Node 3
         ↓
Operation complete
```

---

## Hazelcast vs Other Solutions

| Feature             | Hazelcast                           | Redis           | Memcached       |
| ------------------- | ----------------------------------- | --------------- | --------------- |
| **Clustering**      | Built-in, automatic                 | Manual setup    | No clustering   |
| **Data Structures** | Many (Map, Queue, List, Set, Topic) | Many            | Key-Value only  |
| **Persistence**     | Optional                            | Yes             | No              |
| **Language**        | Java (embedded)                     | Separate server | Separate server |
| **Complexity**      | Low (just add library)              | Medium          | Low             |

---

## When to Use Hazelcast

✅ **Good for:**

- High-speed data access
- Session storage in web apps
- Distributed caching
- Microservices communication
- Real-time data sharing

❌ **Not ideal for:**

- Long-term data storage (use a database)
- Very large datasets that don't fit in RAM
- Single-server applications (overkill)

---

That's Hazelcast! The beauty is that you use it like normal Java collections (`Map`, `Queue`, etc.), but Hazelcast handles all the distribution, replication, and clustering automatically behind the scenes.

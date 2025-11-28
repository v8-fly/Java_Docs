# Customer Support Agent Rating System - Problem Statement

## 📋 **Problem Overview**

You are building a **Customer Support Agent Rating System** for a company that wants to track and analyze the performance of their support agents based on customer feedback.

---

## 🎯 **Functional Requirements**

### **Core Features:**

1. **Agent Management**

   - Add new support agents to the system
   - Each agent has a unique ID and name
   - View all agents in the system

2. **Rating Submission**

   - Customers can submit ratings for agents (scale: 1-5)
   - Each rating must include:
     - Agent ID
     - Rating value (1-5)
     - Date of rating
   - System should validate ratings before storing

3. **Rating Analytics**
   - **Overall Best Agents**: Get agents ranked by their average rating across all time
   - **Monthly Best Agent**: Get the best performing agent for each month
   - All rankings should show agent details and their average rating

### **API Requirements:**

- RESTful APIs for all operations
- Proper HTTP status codes and error handling
- JSON request/response format

---

## 🔧 **Technical Requirements**

1. **Technology Stack:**

   - Spring Boot 3.x
   - Java 17+
   - In-memory storage (H2 Database or simple in-memory repository)
   - Maven/Gradle

2. **Design Patterns to Implement:**

   - Factory Pattern (for creating ratings)
   - Strategy Pattern (for different ranking algorithms)
   - Repository Pattern (for data access)
   - Builder Pattern (for DTOs)

3. **Code Quality:**
   - Proper layering (Controller → Service → Repository)
   - Input validation
   - Exception handling
   - Clean code principles

---

## 📊 **Sample Data & Use Cases**

### **Use Case 1: Add Agents**

```
Agent 1: ID="A001", Name="John Doe"
Agent 2: ID="A002", Name="Jane Smith"
Agent 3: ID="A003", Name="Mike Johnson"
```

### **Use Case 2: Submit Ratings**

```
- Customer rates John (A001): 5 stars on 2024-01-15
- Customer rates Jane (A002): 4 stars on 2024-01-20
- Customer rates John (A001): 4 stars on 2024-02-10
- Customer rates Mike (A003): 5 stars on 2024-02-15
```

### **Use Case 3: Get Overall Rankings**

```
Expected Output:
1. John Doe (A001) - Average: 4.5
2. Mike Johnson (A003) - Average: 5.0
3. Jane Smith (A002) - Average: 4.0
```

### **Use Case 4: Get Monthly Best Agents**

```
Expected Output:
- January 2024: John Doe (A001) - Average: 5.0
- February 2024: Mike Johnson (A003) - Average: 5.0
```

---

## 🚀 **API Endpoints to Build**

```
POST   /api/agents                    - Create a new agent
GET    /api/agents                    - Get all agents
GET    /api/agents/{id}              - Get agent by ID

POST   /api/ratings                   - Submit a rating
GET    /api/ratings                   - Get all ratings
GET    /api/ratings/agent/{agentId}  - Get ratings for specific agent

GET    /api/rankings/overall          - Get overall best agents
GET    /api/rankings/monthly          - Get best agent per month
```

---

## 📦 **Deliverables**

1. ✅ Working Spring Boot application
2. ✅ All API endpoints functional
3. ✅ Proper request/response DTOs
4. ✅ Input validation
5. ✅ Exception handling
6. ✅ Clean project structure
7. ✅ Postman collection (or curl commands) to test

---

## 🎓 **Learning Objectives**

By building this system, you'll learn:

- Spring Boot REST API development
- Design pattern implementation in real projects
- Proper layered architecture
- Bean management and dependency injection
- Exception handling and validation
- Clean code practices

---

# 🏗️ **Let's Build It Step by Step!**

I'll guide you through building this system piece by piece. We'll start with the foundation and gradually add complexity.

## **Step 1: Project Setup** ⚙️

Let's create the Spring Boot project structure. Are you ready to start?

I'll help you with:

1. Creating the project structure
2. Setting up dependencies
3. Building entities first
4. Then repositories
5. Then services with patterns
6. Finally controllers (APIs)
7. Testing everything

**Would you like to:**

- A) Start with project setup and dependencies (pom.xml)
- B) Jump straight to creating the Agent entity
- C) See the complete project structure first

Let me know, and we'll build this together! 🚀

# 🎯 **Great Choice! PostgreSQL + MyBatis is EXCELLENT for this use case**

Let me explain why this is a **solid decision** and how we'll adapt our design:

---

## ✅ **Why PostgreSQL + MyBatis is Good Here**

### **PostgreSQL Benefits:**

1. **Production-Ready**: Real database (not just in-memory H2)
2. **Complex Queries**: Great for aggregations (averages, monthly grouping)
3. **Date Functions**: Excellent support for date operations (monthly rankings)
4. **Scalability**: Can handle millions of ratings
5. **ACID Compliance**: Data integrity for ratings

### **MyBatis Benefits:**

1. **SQL Control**: You write actual SQL (great for learning!)
2. **Performance**: Optimized queries for rankings/aggregations
3. **Flexibility**: Can write complex JOINs and window functions
4. **Clear Separation**: SQL in XML/annotations, not hidden in ORM magic
5. **Interview Appeal**: Shows you understand SQL, not just ORM abstractions

---

## 🏗️ **Updated Architecture with PostgreSQL + MyBatis**

### **Database Schema:**

```sql
-- Agents Table
CREATE TABLE agents (
    agent_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Ratings Table
CREATE TABLE ratings (
    rating_id BIGSERIAL PRIMARY KEY,
    agent_id VARCHAR(50) NOT NULL,
    rating_value INTEGER NOT NULL CHECK (rating_value BETWEEN 1 AND 5),
    rating_date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES agents(agent_id)
);

-- Indexes for performance
CREATE INDEX idx_ratings_agent_id ON ratings(agent_id);
CREATE INDEX idx_ratings_date ON ratings(rating_date);
CREATE INDEX idx_ratings_agent_date ON ratings(agent_id, rating_date);
```

---

## 📂 **Updated Project Structure**

```
src/main/java/com/company/agentrating/
│
├── AgentRatingApplication.java          # Main Spring Boot class
│
├── controller/                          # REST API Layer
│   ├── AgentController.java
│   ├── RatingController.java
│   └── RankingController.java
│
├── service/                             # Business Logic Layer
│   ├── AgentService.java
│   ├── RatingService.java
│   └── RankingService.java
│
├── repository/                          # Data Access Layer
│   ├── AgentMapper.java                 # MyBatis Mapper Interface
│   └── RatingMapper.java                # MyBatis Mapper Interface
│
├── domain/                              # Entities/Models
│   ├── Agent.java
│   └── Rating.java
│
├── dto/                                 # Data Transfer Objects
│   ├── request/
│   │   ├── CreateAgentRequest.java
│   │   └── CreateRatingRequest.java
│   └── response/
│       ├── AgentResponse.java
│       ├── RatingResponse.java
│       ├── OverallRankingResponse.java
│       └── MonthlyRankingResponse.java
│
├── factory/                             # Factory Pattern
│   └── RatingFactory.java
│
├── strategy/                            # Strategy Pattern
│   ├── RankingStrategy.java
│   ├── OverallRankingStrategy.java
│   └── MonthlyRankingStrategy.java
│
├── exception/                           # Exception Handling
│   ├── GlobalExceptionHandler.java
│   ├── AgentNotFoundException.java
│   └── InvalidRatingException.java
│
└── config/                              # Configuration
    └── MyBatisConfig.java

src/main/resources/
│
├── application.yml                      # Spring Boot Config
├── schema.sql                          # Database Schema
├── data.sql                            # Sample Data
│
└── mapper/                             # MyBatis XML Mappers
    ├── AgentMapper.xml
    └── RatingMapper.xml
```

---

## 📦 **Dependencies (pom.xml)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.company</groupId>
    <artifactId>agent-rating-system</artifactId>
    <version>1.0.0</version>
    <name>Customer Support Agent Rating System</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- MyBatis Spring Boot Starter -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>

        <!-- PostgreSQL Driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Lombok (Optional - reduces boilerplate) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Dev Tools (for hot reload) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## ⚙️ **Configuration (application.yml)**

```yaml
spring:
  application:
    name: agent-rating-system

  datasource:
    url: jdbc:postgresql://localhost:5432/agent_rating_db
    username: postgres
    password: your_password
    driver-class-name: org.postgresql.Driver

  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
      data-locations: classpath:data.sql

# MyBatis Configuration
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.company.agentrating.domain
  configuration:
    map-underscore-to-camel-case: true
    default-fetch-size: 100
    default-statement-timeout: 30

# Logging
logging:
  level:
    com.company.agentrating: DEBUG
    org.mybatis: DEBUG

server:
  port: 8080
```

---

## 🎯 **Key Advantages of MyBatis Here**

### **1. Complex Aggregation Queries Made Easy:**

```sql
-- Overall Rankings (in RatingMapper.xml)
SELECT
    a.agent_id,
    a.name,
    AVG(r.rating_value) as average_rating,
    COUNT(r.rating_id) as total_ratings
FROM agents a
LEFT JOIN ratings r ON a.agent_id = r.agent_id
GROUP BY a.agent_id, a.name
ORDER BY average_rating DESC
```

### **2. Monthly Rankings with Window Functions:**

```sql
-- Monthly Rankings (PostgreSQL specific)
WITH monthly_avg AS (
    SELECT
        agent_id,
        DATE_TRUNC('month', rating_date) as month,
        AVG(rating_value) as avg_rating
    FROM ratings
    GROUP BY agent_id, DATE_TRUNC('month', rating_date)
),
ranked AS (
    SELECT
        agent_id,
        month,
        avg_rating,
        ROW_NUMBER() OVER (PARTITION BY month ORDER BY avg_rating DESC) as rank
    FROM monthly_avg
)
SELECT * FROM ranked WHERE rank = 1
```

### **3. Performance Benefits:**

- Direct SQL means you can optimize queries
- Can use PostgreSQL-specific features (EXPLAIN, indexes, etc.)
- Clear view of what queries are running

---

## 🤔 **Potential Concerns & Solutions**

### **Concern 1: "MyBatis has more boilerplate than JPA"**

✅ **True, but worth it here because:**

- You're learning - seeing SQL helps understanding
- Complex aggregations are clearer in SQL
- Better for interviews (shows SQL knowledge)

### **Concern 2: "Setting up is harder"**

✅ **We'll do it step by step:**

- I'll provide all XML mappers
- Configuration is straightforward
- Once set up, very maintainable

### **Concern 3: "Is it modern?"**

✅ **Absolutely:**

- MyBatis 3.x is actively maintained
- Used by Alibaba, Netflix, and many others
- Great for microservices with complex queries

---

## 🚀 **Our Building Plan**

I'll guide you through in this order:

1. **PostgreSQL Setup** (5 min)

   - Install/verify PostgreSQL
   - Create database
   - Run schema

2. **Domain Models** (10 min)

   - Agent.java
   - Rating.java

3. **MyBatis Mappers** (20 min)

   - AgentMapper interface + XML
   - RatingMapper interface + XML

4. **Services with Patterns** (30 min)

   - Factory Pattern for Rating creation
   - Strategy Pattern for ranking algorithms

5. **REST Controllers** (20 min)

   - All API endpoints

6. **Testing** (15 min)
   - Postman/curl commands

---

## ✅ **Final Verdict: PostgreSQL + MyBatis**

**Perfect for this project because:**

- ✅ Real-world production setup
- ✅ Complex aggregations (rankings) are natural in SQL
- ✅ Shows SQL expertise (interview bonus)
- ✅ Clean separation of concerns
- ✅ Performance optimization possible
- ✅ Great learning experience

---

# 🎬 **Ready to Start?**

**Let's begin! What would you like to do first?**

**Option A**: Set up PostgreSQL database and verify connection
**Option B**: Start coding the domain models (Agent, Rating)
**Option C**: See the complete MyBatis mapper examples first

Just say "A", "B", or "C" and we'll dive in! 🚀

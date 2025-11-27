Absolutely! Let me break down the MyBatis basic example step by step.

## The Complete Example Explained

Let's build a simple application that manages users in a database.

---

## Step 1: The Java Entity Class

First, we need a simple Java class to represent our data:

```java
public class User {
    private Long id;
    private String username;
    private String email;
    private Date createdDate;

    // Constructors
    public User() {}

    public User(String username, String email) {
        this.username = username;
        this.email = email;
        this.createdDate = new Date();
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public Date getCreatedDate() { return createdDate; }
    public void setCreatedDate(Date createdDate) { this.createdDate = createdDate; }
}
```

**What this is:** Just a regular Java class (POJO) that represents a row in our `users` table.

---

## Step 2: The Database Table

Our database has a table like this:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**What this is:** The actual database table where user data is stored.

---

## Step 3: The Mapper Interface

Now we create an interface that defines what database operations we want:

```java
package com.example.mapper;

import com.example.model.User;
import java.util.List;

public interface UserMapper {
    // Find a single user by ID
    User findById(Long id);

    // Get all users
    List<User> findAll();

    // Insert a new user
    void insertUser(User user);

    // Update an existing user
    void updateUser(User user);

    // Delete a user
    void deleteUser(Long id);
}
```

**What this is:**

- Just an **interface** (no implementation code!)
- Each method represents a database operation
- MyBatis will automatically create the implementation for us
- **Important:** This is just a contract - it says "these are the operations I want to do"

---

## Step 4: The XML Mapper File

This is where the **magic happens**. We connect our Java methods to actual SQL queries:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- The namespace MUST match your interface's full class name -->
<mapper namespace="com.example.mapper.UserMapper">

    <!-- This matches the findById method -->
    <select id="findById" resultType="com.example.model.User">
        SELECT id, username, email, created_date
        FROM users
        WHERE id = #{id}
    </select>

    <!-- This matches the findAll method -->
    <select id="findAll" resultType="com.example.model.User">
        SELECT id, username, email, created_date
        FROM users
        ORDER BY username
    </select>

    <!-- This matches the insertUser method -->
    <insert id="insertUser" parameterType="com.example.model.User">
        INSERT INTO users (username, email, created_date)
        VALUES (#{username}, #{email}, #{createdDate})
    </insert>

    <!-- This matches the updateUser method -->
    <update id="updateUser" parameterType="com.example.model.User">
        UPDATE users
        SET username = #{username},
            email = #{email}
        WHERE id = #{id}
    </update>

    <!-- This matches the deleteUser method -->
    <delete id="deleteUser" parameterType="Long">
        DELETE FROM users WHERE id = #{id}
    </delete>

</mapper>
```

**Let's break down each part:**

### The `<mapper>` tag

```xml
<mapper namespace="com.example.mapper.UserMapper">
```

- `namespace`: **Must match** your interface's full package + class name
- This is how MyBatis connects the XML to your Java interface

### The `<select>` tag (for queries that return data)

```xml
<select id="findById" resultType="com.example.model.User">
    SELECT id, username, email, created_date
    FROM users
    WHERE id = #{id}
</select>
```

- `id="findById"`: **Must match** the method name in your interface
- `resultType`: What type of object to return (our User class)
- `#{id}`: A **parameter placeholder** - MyBatis replaces this with the actual value passed to the method
- The SQL is what you would normally write

### The `<insert>` tag

```xml
<insert id="insertUser" parameterType="com.example.model.User">
    INSERT INTO users (username, email, created_date)
    VALUES (#{username}, #{email}, #{createdDate})
</insert>
```

- `parameterType`: The type of object we're inserting (our User object)
- `#{username}`: MyBatis calls `user.getUsername()` automatically
- `#{email}`: MyBatis calls `user.getEmail()` automatically
- `#{createdDate}`: MyBatis calls `user.getCreatedDate()` automatically

---

## Step 5: MyBatis Configuration

We need to configure MyBatis. This file is typically named `mybatis-config.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- Database connection settings -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>

    <!-- Tell MyBatis where to find our mapper XML files -->
    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

**What this does:**

- Sets up database connection details
- Tells MyBatis where to find our mapper XML files

---

## Step 6: Using MyBatis in Your Application

Now let's actually use it:

```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;
import java.util.List;

public class MyBatisExample {

    public static void main(String[] args) throws Exception {

        // STEP 1: Load MyBatis configuration and build SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory =
            new SqlSessionFactoryBuilder().build(inputStream);

        // STEP 2: Open a session (like opening a database connection)
        SqlSession session = sqlSessionFactory.openSession();

        try {
            // STEP 3: Get the mapper implementation
            // MyBatis creates the implementation automatically!
            UserMapper mapper = session.getMapper(UserMapper.class);

            // STEP 4: Use the mapper methods

            // INSERT a new user
            System.out.println("=== Inserting a new user ===");
            User newUser = new User("john_doe", "john@example.com");
            mapper.insertUser(newUser);
            session.commit(); // Don't forget to commit!
            System.out.println("User inserted!");

            // FIND ALL users
            System.out.println("\n=== Finding all users ===");
            List<User> allUsers = mapper.findAll();
            for (User user : allUsers) {
                System.out.println("ID: " + user.getId() +
                                 ", Username: " + user.getUsername() +
                                 ", Email: " + user.getEmail());
            }

            // FIND ONE user by ID
            System.out.println("\n=== Finding user with ID = 1 ===");
            User user = mapper.findById(1L);
            if (user != null) {
                System.out.println("Found: " + user.getUsername());
            }

            // UPDATE a user
            System.out.println("\n=== Updating user ===");
            user.setEmail("newemail@example.com");
            mapper.updateUser(user);
            session.commit();
            System.out.println("User updated!");

            // DELETE a user
            System.out.println("\n=== Deleting user with ID = 1 ===");
            mapper.deleteUser(1L);
            session.commit();
            System.out.println("User deleted!");

        } finally {
            // STEP 5: Always close the session
            session.close();
        }
    }
}
```

**Let's break down what happens:**

### Step 1: Build SqlSessionFactory

```java
SqlSessionFactory sqlSessionFactory =
    new SqlSessionFactoryBuilder().build(inputStream);
```

- Reads the `mybatis-config.xml` file
- Sets up MyBatis with database connection info
- Creates a factory that will give us sessions
- **Do this ONCE** in your application (usually at startup)

### Step 2: Open a Session

```java
SqlSession session = sqlSessionFactory.openSession();
```

- Opens a connection to the database
- Think of it like opening a JDBC connection
- **Must be closed** when done (use try-finally or try-with-resources)

### Step 3: Get the Mapper

```java
UserMapper mapper = session.getMapper(UserMapper.class);
```

- **This is the magic!**
- MyBatis creates an implementation of your interface automatically
- You never write the implementation code yourself
- The mapper knows to use the SQL from your XML file

### Step 4: Use the Mapper

```java
User user = mapper.findById(1L);
```

**What happens behind the scenes:**

1. You call `mapper.findById(1L)`
2. MyBatis looks in the XML for `<select id="findById">`
3. MyBatis takes your parameter `1L` and replaces `#{id}` with it
4. MyBatis executes: `SELECT id, username, email, created_date FROM users WHERE id = 1`
5. MyBatis gets the result from the database
6. MyBatis creates a new `User` object
7. MyBatis calls `user.setId()`, `user.setUsername()`, etc. with the values from the database
8. MyBatis returns the `User` object to you

### Step 5: Commit and Close

```java
session.commit(); // Save changes to database
session.close();  // Release database connection
```

- **Important:** INSERT, UPDATE, DELETE need `commit()` or changes won't be saved
- Always close the session to avoid connection leaks

---

## The Flow Diagram

```
Your Code
    ↓
mapper.findById(1L)  ← You call the interface method
    ↓
MyBatis looks at UserMapper.xml
    ↓
Finds: <select id="findById">
    ↓
Replaces #{id} with 1
    ↓
Executes SQL: SELECT ... WHERE id = 1
    ↓
Database returns data
    ↓
MyBatis creates User object
    ↓
MyBatis sets properties using setters
    ↓
Returns User object to your code
```

---

## Key Takeaways

1. **You write the interface** (UserMapper.java) - just method signatures
2. **You write the SQL** (UserMapper.xml) - the actual queries
3. **MyBatis connects them** - through matching namespace and id
4. **MyBatis creates the implementation** - you never write it yourself
5. **You use it like any normal Java object** - call methods, get results

The beauty of MyBatis is that you have **full control over SQL** but don't have to write the boring JDBC boilerplate code (opening connections, creating statements, looping through result sets, etc.).

# **`@Transactional` Annotation in Spring Boot**  

## **1️⃣ What is `@Transactional` in Spring Boot?**
`@Transactional` is a **Spring annotation** that **manages database transactions automatically**. It ensures that:
✅ **All operations inside a method are executed as a single unit (Atomicity)**.  
✅ **If an error occurs, all changes are rolled back (Consistency)**.  
✅ **Ensures data integrity in case of failures (Rollback on Exception)**.  

👉 It is used in **Spring Data JPA**, **Hibernate**, and **JDBC-based applications**.

---

## **2️⃣ Why Use `@Transactional`?**
- **Ensures Atomicity** → If one operation fails, all other operations within the transaction are **rolled back**.
- **Manages Connections Efficiently** → Handles database connections and commits automatically.
- **Prevents Partial Updates** → Avoids data corruption if an error occurs mid-execution.

---

## **3️⃣ Basic Example: Using `@Transactional`**
### **Scenario: Saving Two Users in a Transaction**
```java
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUsers() {
        User user1 = new User("Alice", "alice@example.com");
        User user2 = new User("Bob", "bob@example.com");

        userRepository.save(user1);
        userRepository.save(user2);
    }
}
```
✅ If **both `save()` calls succeed**, data is saved.  
❌ If **one `save()` fails**, both inserts are rolled back.

---

## **4️⃣ Automatic Rollback in `@Transactional`**
If an **unchecked exception** (`RuntimeException`) occurs, the transaction **rolls back automatically**.

### **Example: Simulating an Exception**
```java
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUsers() {
        User user1 = new User("Alice", "alice@example.com");
        userRepository.save(user1);

        // Simulating an error (division by zero)
        int result = 1 / 0;  

        User user2 = new User("Bob", "bob@example.com");
        userRepository.save(user2);
    }
}
```
👉 **Transaction behavior**:
- ✅ `user1` is **saved** in the database.
- ❌ `user2` is **not saved** because an exception occurs.
- 🔄 **Rollback happens** → `user1` is also removed.

---

## **5️⃣ Handling Rollback for Checked Exceptions**
By default, `@Transactional` **rolls back only on `RuntimeException` and `Error`**.  
👉 If you want to roll back on **checked exceptions** (e.g., `SQLException`), specify `rollbackFor`.

### **Example: Rollback on Checked Exception**
```java
import jakarta.transaction.Transactional;
import java.sql.SQLException;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional(rollbackFor = SQLException.class)
    public void createUsers() throws SQLException {
        User user1 = new User("Alice", "alice@example.com");
        userRepository.save(user1);

        throw new SQLException("Database error!");

        User user2 = new User("Bob", "bob@example.com");
        userRepository.save(user2);
    }
}
```
✅ Now, `@Transactional` **rolls back** on `SQLException`.  

---

## **6️⃣ Preventing Rollback for Specific Exceptions**
You can **ignore certain exceptions** using `noRollbackFor`.

### **Example: No Rollback for `NullPointerException`**
```java
import jakarta.transaction.Transactional;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional(noRollbackFor = NullPointerException.class)
    public void createUsers() {
        User user1 = new User("Alice", "alice@example.com");
        userRepository.save(user1);

        throw new NullPointerException("Some minor issue!");

        User user2 = new User("Bob", "bob@example.com");
        userRepository.save(user2);
    }
}
```
✅ **Transaction behavior**:
- ❌ `NullPointerException` **does not trigger rollback**.
- ✅ `user1` **remains saved** in the database.

---

## **7️⃣ Using `@Transactional` at Class Level**
Instead of applying `@Transactional` on methods, you can apply it at the **class level**.

```java
import jakarta.transaction.Transactional;
import org.springframework.stereotype.Service;

@Service
@Transactional  // Applies to all methods in this class
public class UserService {
    
    @Autowired
    private UserRepository userRepository;

    public void createUser() {
        userRepository.save(new User("Alice", "alice@example.com"));
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```
✅ All methods **inherit transactional behavior**.

---

## **8️⃣ Propagation in Transactions (`propagation` Attribute)**
Propagation defines **how transactions behave** when called inside another transaction.

| Propagation Type | Behavior |
|------------------|----------|
| `REQUIRED` (Default) | Uses an **existing transaction** or creates a new one if none exists. |
| `REQUIRES_NEW` | **Always creates a new transaction**, suspending the existing one. |
| `NESTED` | Creates a **nested transaction** inside an existing one. |
| `SUPPORTS` | Runs inside a transaction **if one exists**, otherwise runs without one. |
| `NOT_SUPPORTED` | Runs **without** a transaction, suspending any existing one. |
| `MANDATORY` | Requires an existing transaction, **throws an exception if none exists**. |

---

### **Example: `REQUIRES_NEW`**
```java
import jakarta.transaction.Transactional;

@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUsers() {
        User user1 = new User("Alice", "alice@example.com");
        userRepository.save(user1);

        createUserWithNewTransaction();

        User user2 = new User("Bob", "bob@example.com");
        userRepository.save(user2);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createUserWithNewTransaction() {
        User user = new User("Charlie", "charlie@example.com");
        userRepository.save(user);
    }
}
```
✅ **Transaction Behavior**:
- `createUsers()` → Runs inside **Transaction 1**.
- `createUserWithNewTransaction()` → Runs inside **Transaction 2 (separate transaction)**.
- If `createUsers()` **fails**, `Charlie` is **not rolled back** because it's in a different transaction.

---

## **9️⃣ Transaction Isolation Levels**
Isolation levels control **how concurrent transactions interact**.

| Isolation Level | Behavior |
|----------------|----------|
| `READ_COMMITTED` (Default) | Prevents reading **uncommitted** data. |
| `READ_UNCOMMITTED` | Allows reading **uncommitted** data (dirty reads). |
| `REPEATABLE_READ` | Prevents **non-repeatable reads** (data changes mid-transaction). |
| `SERIALIZABLE` | Fully isolates transactions (slower but safest). |

---

### **Example: Setting Isolation Level**
```java
import jakarta.transaction.Transactional;
import org.springframework.transaction.annotation.Isolation;

@Service
public class UserService {

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void processData() {
        // Read and update data with repeatable read isolation
    }
}
```
✅ **Prevents changes to data while the transaction is in progress**.

---

## **🔹 Best Practices for `@Transactional`**
✅ Apply `@Transactional` only on **service layer** methods.  
✅ Use **`rollbackFor`** to roll back on **checked exceptions**.  
✅ Be careful when using **`REQUIRES_NEW`**, as it creates **separate transactions**.  
✅ Use **fine-grained transaction control** instead of marking everything `@Transactional`.  

---

## **🚀 Summary of `@Transactional`**
| Feature | Description |
|---------|------------|
| **Atomicity** | Ensures all database operations inside a transaction are executed as one unit. |
| **Rollback Behavior** | Rolls back automatically on `RuntimeException`. |
| **Rollback for Checked Exceptions** | Use `rollbackFor` to roll back on checked exceptions. |
| **Propagation** | Controls how transactions behave inside other transactions. |
| **Isolation Levels** | Defines how concurrent transactions interact. |

---

## **🚀 Conclusion**
- `@Transactional` simplifies **database transaction management**.
- It ensures **data consistency** by **rolling back on failures**.
- You can **customize rollback behavior**, **propagation**, and **isolation levels**.

Would you like an **example with multiple database transactions**? 😊
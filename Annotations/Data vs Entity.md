## **Difference Between `@Data` and `@Entity` Annotations in Spring Boot**

### **1. `@Data` (Lombok)**
The `@Data` annotation comes from **Lombok**, a Java library that automatically generates **boilerplate code** like getters, setters, `toString()`, `equals()`, and `hashCode()` methods for your class.

✅ **Key Features of `@Data`:**  
- Automatically generates:
  - **Getters & Setters**
  - **`toString()`**
  - **`equals()` & `hashCode()`**
  - **A default constructor (if no other constructors exist)**
- Reduces **boilerplate code** in POJOs (Plain Old Java Objects).
- Used mainly in **DTOs (Data Transfer Objects)** and **model classes**.

#### **Example Using `@Data`**
```java
import lombok.Data;

@Data
public class UserDTO {
    private String name;
    private String email;
}
```
👉 **No need to manually write getters and setters!**  
👉 Lombok **automatically** generates `getName()`, `setName()`, `getEmail()`, `setEmail()`, and `toString()`.  

---

### **2. `@Entity` (JPA)**
The `@Entity` annotation comes from **Jakarta Persistence API (JPA)** and is used to mark a **Java class as a database entity**. This class will be **mapped to a database table** by Hibernate (or another JPA provider).

✅ **Key Features of `@Entity`:**  
- Marks the class as **a persistent entity**.
- Maps the class to a **table in a relational database**.
- Requires a **primary key (`@Id`)**.
- Used in **JPA repositories** for database interactions.

#### **Example Using `@Entity`**
```java
import jakarta.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Getters and setters
}
```
👉 This `User` class is mapped to a **database table**.  
👉 Each instance of `User` is a **record in the database**.

---

## **Key Differences Between `@Data` and `@Entity`**
| Feature | `@Data` (Lombok) | `@Entity` (JPA) |
|---------|----------------|----------------|
| **Purpose** | Generates boilerplate code (getters, setters, `toString()`, etc.) | Marks a class as a JPA entity for database persistence |
| **Belongs to** | Lombok (`lombok.Data`) | JPA (`jakarta.persistence.Entity`) |
| **Main Use Case** | DTOs, model classes | Database entities (tables) |
| **Requires Primary Key?** | ❌ No | ✅ Yes (`@Id` required) |
| **Persistence** | ❌ No database interaction | ✅ Stored in a database |

---

## **Can You Use `@Data` and `@Entity` Together?**
Yes, but **with caution!**  

Using `@Data` on an `@Entity` class can cause issues because Lombok generates **`equals()` and `hashCode()` methods**, which may lead to **Hibernate performance problems** (e.g., infinite loops in bidirectional relationships).

### **Best Practice: Use `@Getter` and `@Setter` Instead of `@Data`**
```java
import lombok.Getter;
import lombok.Setter;
import jakarta.persistence.*;

@Entity
@Getter
@Setter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;
}
```
✅ **This avoids Hibernate issues while still reducing boilerplate code.**

---

## **Conclusion**
- Use `@Data` for **DTOs, model classes, and non-persistent objects**.
- Use `@Entity` for **JPA entities** that need to be stored in the database.
- Avoid `@Data` with `@Entity` due to potential **Hibernate issues**; instead, use `@Getter` and `@Setter`.

Would you like an example of how to use **DTOs with JPA entities**? 😊
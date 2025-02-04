# **`@Retry` Annotation in Spring Boot**  
The `@Retry` annotation is used in **Spring Boot with Resilience4j** to **automatically retry a failed operation** (such as a failed API call, database transaction, or remote service call). It helps improve **fault tolerance** by **re-attempting** the failed method execution before giving up.

---

## **1️⃣ How Does `@Retry` Work?**
- If a method **fails**, Spring will automatically **retry** the method execution based on the configured retry settings.
- You can define:
  - **Number of retries**
  - **Retry interval (time delay between retries)**
  - **Exception conditions (which exceptions should trigger a retry)**

✅ Useful in **microservices architecture**, **remote API calls**, **database operations**, and **network failures**.

---

## **2️⃣ Adding Dependencies**
To use `@Retry`, you need **Resilience4j** dependencies.

### **For Maven:**
```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>2.0.2</version>
</dependency>
```

### **For Gradle:**
```gradle
implementation 'io.github.resilience4j:resilience4j-spring-boot2:2.0.2'
```

---

## **3️⃣ Enable Retry in Spring Boot**
Enable Resilience4j retrying by adding `@EnableRetry` in your Spring Boot main class:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry  // Enables Retry Mechanism
public class RetryExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(RetryExampleApplication.class, args);
    }
}
```
---

## **4️⃣ Basic Example of `@Retry`**
### **Simulating a Failed API Call**
Let's create a service that **fails randomly** and then automatically **retries** using `@Retry`.

```java
import io.github.resilience4j.retry.annotation.Retry;
import org.springframework.stereotype.Service;
import java.util.Random;

@Service
public class ApiService {
    
    private static final Random random = new Random();

    @Retry(name = "apiService", fallbackMethod = "fallbackResponse")
    public String fetchData() {
        if (random.nextBoolean()) {  // Simulating failure randomly
            throw new RuntimeException("API request failed!");
        }
        return "Data from API";
    }

    public String fallbackResponse(Exception ex) {
        return "Fallback: Default Data";
    }
}
```
### **Explanation**
- `@Retry(name = "apiService", fallbackMethod = "fallbackResponse")`
  - If `fetchData()` fails, it **automatically retries** using Resilience4j.
  - If retries **exceed the limit**, it calls `fallbackResponse()`.
- `fallbackResponse()` provides **default behavior** when retries fail.

---

## **5️⃣ Configuring Retry Settings**
Define retry settings in `application.yml` or `application.properties`.

### **`application.yml`**
```yaml
resilience4j.retry:
  instances:
    apiService:
      max-attempts: 3
      wait-duration: 2s
      retry-exceptions:
        - java.io.IOException
        - java.util.concurrent.TimeoutException
      ignore-exceptions:
        - java.lang.NullPointerException
```
### **Explanation**
- `max-attempts: 3` → Retry **3 times** before failing.
- `wait-duration: 2s` → Wait **2 seconds** between retries.
- `retry-exceptions` → Retry only for these exceptions.
- `ignore-exceptions` → Do **not retry** for these exceptions.

---

## **6️⃣ Using `@Retry` in a REST Controller**
Create a REST API that retries failed calls.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class ApiController {

    @Autowired
    private ApiService apiService;

    @GetMapping("/fetch")
    public String fetchData() {
        return apiService.fetchData();
    }
}
```
**Test the API (`http://localhost:8080/api/fetch`)**:
- If the first attempt **fails**, it **retries** up to 3 times.
- If retries **exceed the limit**, it returns the **fallback response**.

---

## **7️⃣ Handling Specific Exceptions**
You can specify **which exceptions** should trigger a retry.

```java
@Retry(name = "dbService", fallbackMethod = "databaseFallback")
public String fetchDatabaseData() throws SQLException {
    throw new SQLException("Database connection failed!");
}
public String databaseFallback(SQLException ex) {
    return "Fallback: Default Database Data";
}
```
✅ Here, `fetchDatabaseData()` **retries only for `SQLException`**.

---

## **8️⃣ Combining `@Retry` with `@CircuitBreaker`**
- **`@Retry`** retries failures **immediately**.
- **`@CircuitBreaker`** prevents failures from overloading the system.

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;

@Retry(name = "retryService", fallbackMethod = "fallbackResponse")
@CircuitBreaker(name = "circuitService", fallbackMethod = "fallbackResponse")
public String fetchData() {
    throw new RuntimeException("Service failed!");
}

public String fallbackResponse(Exception ex) {
    return "Circuit Breaker & Retry Fallback";
}
```
✅ If the service **fails multiple times**, the circuit breaker **opens** to stop further requests.

---

## **9️⃣ `@Retry` vs `@CircuitBreaker`**
| Feature | `@Retry` | `@CircuitBreaker` |
|---------|---------|----------------|
| **Purpose** | Retries failed requests | Stops excessive failures to prevent overload |
| **When Applied** | Immediately after failure | After multiple failures in a short period |
| **Fallback** | Can provide a fallback method | Can provide a fallback method |
| **Use Case** | Unstable APIs, databases | Preventing system overload |

---

## **🔹 When to Use `@Retry`?**
✅ **Remote API Calls** – Retrying failed HTTP requests.  
✅ **Database Transactions** – Retrying database operations that fail due to **transient errors**.  
✅ **Network Failures** – Handling intermittent network issues.  
✅ **Rate-Limited APIs** – Retrying after **rate limits** expire.  

---

## **🔹 When NOT to Use `@Retry`?**
🚫 **Permanent Failures** – If an error **won't resolve with retries**, it's better to **fail fast**.  
🚫 **Idempotent Operations** – Avoid retrying non-idempotent operations (**like money transfers**).  
🚫 **Long Wait Times** – Too many retries can **increase response times**.  

---

## **🔹 Summary of `@Retry`**
| Feature | Description |
|---------|------------|
| **Annotation** | `@Retry(name = "serviceName", fallbackMethod = "fallback")` |
| **Purpose** | Retries failed method execution before failing |
| **Exception Handling** | Retry on **specific exceptions**, ignore others |
| **Configuration** | Customize retry attempts, delay, and exceptions via `application.yml` |
| **Fallback Method** | A method that runs if all retries fail |

---

## **🚀 Conclusion**
- `@Retry` **automatically retries** failed method executions before failing.
- Works **best for temporary failures** like **network issues** and **database timeouts**.
- Can be **configured using `application.yml`** for retry attempts and delays.
- Use with **`@CircuitBreaker`** to prevent overloading the system.

---

Would you like a **working Spring Boot project with Resilience4j** for testing? 😊
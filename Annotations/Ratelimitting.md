# **`@RateLimiter` in Spring Boot (Resilience4j)**  
### **Definition**
`@RateLimiter` is an annotation from **Resilience4j** that **limits the number of requests** within a specified time frame. It helps **prevent excessive traffic** from overwhelming a system by **restricting how often a method can be called**.

✅ **Use Case:**  
- Protecting APIs from **excessive calls (e.g., DDoS attacks, API abuse)**.  
- Preventing **backend system overload**.  
- Enforcing **fair usage policies** in **microservices** and **REST APIs**.  

---

## **1️⃣ Adding Dependencies**
To use `@RateLimiter`, add **Resilience4j dependencies** in your Spring Boot project.

### **For Maven**
```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>2.0.2</version>
</dependency>
```

### **For Gradle**
```gradle
implementation 'io.github.resilience4j:resilience4j-spring-boot2:2.0.2'
```

---

## **2️⃣ Enabling Resilience4j in Spring Boot**
Enable Resilience4j support in the **Spring Boot main class**.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RateLimiterApplication {
    public static void main(String[] args) {
        SpringApplication.run(RateLimiterApplication.class, args);
    }
}
```
---
## **3️⃣ Applying `@RateLimiter` to a Service**
### **Example: Limiting Requests to an API**
```java
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import org.springframework.stereotype.Service;

@Service
public class ApiService {

    @RateLimiter(name = "apiService", fallbackMethod = "fallbackResponse")
    public String fetchData() {
        return "Data fetched successfully!";
    }

    public String fallbackResponse(Exception ex) {
        return "Too many requests! Please try again later.";
    }
}
```
✅ If too many requests **exceed the configured limit**, the `fallbackResponse()` method is called.

---

## **4️⃣ Configuring Rate Limiting in `application.yml`**
Define **rate-limiting rules** for `apiService`.

```yaml
resilience4j.ratelimiter:
  instances:
    apiService:
      limit-for-period: 5
      limit-refresh-period: 10s
      timeout-duration: 2s
```

### **Explanation**
- **`limit-for-period: 5`** → Allows **5 requests** per time window.
- **`limit-refresh-period: 10s`** → Every **10 seconds**, the limit resets.
- **`timeout-duration: 2s`** → If a request exceeds the limit, it waits **2 seconds** before failing.

---

## **5️⃣ Using `@RateLimiter` in a REST Controller**
Expose a **rate-limited API**.

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
✅ If **more than 5 requests** are made within **10 seconds**, a **fallback response** is returned.

---

## **6️⃣ Handling Fallbacks**
If a request **exceeds the rate limit**, the **fallback method** is executed.

```java
public String fallbackResponse(Exception ex) {
    return "Too many requests! Please wait.";
}
```
✅ Instead of throwing an error, **a user-friendly message is returned**.

---

## **7️⃣ Customizing Rate Limiting Further**
### **1. Waiting Instead of Failing (`timeout-duration`)**
Instead of failing immediately, configure **a timeout before retrying**.

```yaml
resilience4j.ratelimiter:
  instances:
    apiService:
      limit-for-period: 5
      limit-refresh-period: 10s
      timeout-duration: 5s  # Waits 5 seconds before failing
```

### **2. Setting Different Limits for Different APIs**
You can set different rate limits for **different API endpoints**.

```yaml
resilience4j.ratelimiter:
  instances:
    userService:
      limit-for-period: 10
      limit-refresh-period: 15s
    orderService:
      limit-for-period: 3
      limit-refresh-period: 5s
```
✅ This ensures **user-related APIs get higher priority than order APIs**.

---

## **8️⃣ Combining `@RateLimiter` with `@Retry` and `@CircuitBreaker`**
### **Scenario: If API fails due to rate limiting, retry before failing**
```java
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import io.github.resilience4j.retry.annotation.Retry;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class ApiService {

    @RateLimiter(name = "apiService", fallbackMethod = "fallbackResponse")
    @Retry(name = "retryService", fallbackMethod = "fallbackResponse")
    @CircuitBreaker(name = "circuitService", fallbackMethod = "fallbackResponse")
    public String fetchData() {
        return "Data from API";
    }

    public String fallbackResponse(Exception ex) {
        return "Too many requests! Please wait.";
    }
}
```
✅ **Retries first before rate-limiting takes effect**.

---

## **9️⃣ When to Use `@RateLimiter`?**
✅ **APIs that receive high traffic** (to prevent excessive calls).  
✅ **Microservices communication** (to prevent one service from overwhelming another).  
✅ **Third-party API calls** (to prevent exceeding API limits).  

---

## **🔹 When NOT to Use `@RateLimiter`**
🚫 If your system **can handle high load** without performance issues.  
🚫 For **database operations** (use **bulk processing** instead).  

---

## **🔹 Summary of `@RateLimiter`**
| Feature | Description |
|---------|------------|
| **Annotation** | `@RateLimiter(name = "service", fallbackMethod = "fallback")` |
| **Purpose** | Limits the number of method executions per time period |
| **Configuration** | `limit-for-period`, `limit-refresh-period`, `timeout-duration` |
| **Fallback Handling** | Custom fallback method if the limit is exceeded |
| **Use Case** | API protection, rate limiting microservices, avoiding excessive requests |

---

## **🚀 Conclusion**
- `@RateLimiter` helps **prevent API overuse and excessive calls**.
- It is **easily configurable** via `application.yml`.
- Supports **fallback methods** to provide user-friendly responses.
- Can be **combined with `@Retry` and `@CircuitBreaker`** for better resilience.

---

### **Next Steps**
Would you like a **Spring Boot project with Resilience4j RateLimiter** for testing? 😊
# **`@EnableConfigurationProperties` in Spring Boot**

### **Definition**
The `@EnableConfigurationProperties` annotation is used in Spring Boot to **bind external configuration properties (from `application.properties` or `application.yml`) to Java classes**. It enables and registers **`@ConfigurationProperties`** annotated beans in the Spring context.

---

## **Why Use `@EnableConfigurationProperties`?**
✅ **Simplifies configuration management** by mapping properties to a class.  
✅ **Type-safe binding** (ensures correct data types for properties).  
✅ **Eliminates manual property retrieval** via `@Value`.  
✅ **Supports nested properties and default values**.  

---

## **1️⃣ Basic Example of `@EnableConfigurationProperties`**

### **Step 1: Add Configuration in `application.properties`**
```properties
app.name=MyApp
app.version=1.0.0
app.description=A simple Spring Boot application
```

---

### **Step 2: Create a Configuration Class**
Use `@ConfigurationProperties` to **map properties to a Java class**.

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app")  // Maps properties with "app." prefix
public class AppProperties {

    private String name;
    private String version;
    private String description;

    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
}
```
✅ The properties (`app.name`, `app.version`, `app.description`) are automatically mapped to the `AppProperties` class.

---

### **Step 3: Enable `@EnableConfigurationProperties` in the Main Class**
In Spring Boot 2.2+, **this step is optional if you use `@Component` in `AppProperties`**, but for explicit configuration, you can enable it.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)  // Enables properties binding
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```
✅ `@EnableConfigurationProperties(AppProperties.class)` ensures that Spring Boot registers `AppProperties` as a bean and binds values.

---

### **Step 4: Use the Configuration Properties in a Service**
Inject `AppProperties` into any Spring-managed bean.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class AppService {

    private final AppProperties appProperties;

    @Autowired
    public AppService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public void printAppInfo() {
        System.out.println("App Name: " + appProperties.getName());
        System.out.println("Version: " + appProperties.getVersion());
        System.out.println("Description: " + appProperties.getDescription());
    }
}
```
✅ Spring Boot **automatically injects property values** into `AppProperties`, making them available in the `AppService`.

---

## **2️⃣ Nested Configuration Properties Example**
You can **group related properties** using nested objects.

### **Step 1: Define Properties in `application.yml`**
```yaml
app:
  name: MyApp
  security:
    enabled: true
    secretKey: abc123
```

---

### **Step 2: Create a Class with Nested Properties**
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private Security security = new Security();

    public static class Security {
        private boolean enabled;
        private String secretKey;

        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }

        public String getSecretKey() { return secretKey; }
        public void setSecretKey(String secretKey) { this.secretKey = secretKey; }
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Security getSecurity() { return security; }
    public void setSecurity(Security security) { this.security = security; }
}
```
✅ Now, `app.security.enabled` and `app.security.secretKey` are mapped inside the `Security` nested class.

---

## **3️⃣ Using Default Values**
Spring Boot allows **default values** for properties.

```java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name = "DefaultApp";  // Default value

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```
✅ If `app.name` is missing in `application.properties`, **"DefaultApp"** is used.

---

## **4️⃣ Validating Configuration Properties**
Spring Boot allows **validation** using `@Validated`.

### **Example: Enforcing Non-Empty Property**
```java
import jakarta.validation.constraints.NotBlank;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@Validated
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    @NotBlank  // Ensures this property is not empty or null
    private String name;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```
✅ If `app.name` is missing or empty, Spring Boot will **fail on startup** with a validation error.

---

## **5️⃣ Binding Lists and Maps**
You can **bind lists and maps** to properties.

### **Example: List of Strings (`application.yml`)**
```yaml
app:
  supported-languages:
    - en
    - fr
    - es
```

### **Java Class**
```java
import java.util.List;

@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private List<String> supportedLanguages;

    public List<String> getSupportedLanguages() { return supportedLanguages; }
    public void setSupportedLanguages(List<String> supportedLanguages) {
        this.supportedLanguages = supportedLanguages;
    }
}
```
✅ `supportedLanguages` will contain `["en", "fr", "es"]`.

---

### **Example: Map Binding (`application.yml`)**
```yaml
app:
  database:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
```

### **Java Class**
```java
import java.util.Map;

@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private Map<String, String> database;

    public Map<String, String> getDatabase() { return database; }
    public void setDatabase(Map<String, String> database) { this.database = database; }
}
```
✅ Now, `database.get("url")` returns `jdbc:mysql://localhost:3306/mydb`.

---

## **6️⃣ Difference Between `@Value` and `@ConfigurationProperties`**
| Feature                 | `@Value` | `@ConfigurationProperties` |
|-------------------------|---------|----------------------------|
| **Binding type**        | Single property injection | Binds multiple properties as a POJO |
| **Supports nested values?** | ❌ No | ✅ Yes |
| **Type Safety**         | ❌ No automatic validation | ✅ Supports type safety |
| **Use case**            | Injecting simple values | Binding structured configuration |

### **Example Using `@Value` (Not Recommended for Complex Configs)**
```java
@Value("${app.name}")
private String appName;
```
✅ Use `@Value` for **one-off property injection** but prefer `@ConfigurationProperties` for structured configuration.

---

## **Conclusion**
- `@EnableConfigurationProperties` **enables property binding** to Java classes.
- **Automatically maps `application.properties` or `application.yml` values** to fields in a POJO.
- Supports **lists, maps, nested objects, and validation**.
- **Better than `@Value`** for structured configuration.

Would you like to see an example integrating **Spring Profiles** with `@EnableConfigurationProperties`? 😊
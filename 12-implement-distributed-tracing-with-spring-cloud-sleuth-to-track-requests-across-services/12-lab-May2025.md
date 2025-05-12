
# **Lab 12: Implement Distributed Tracing with Spring Boot 3.4.5 Using Micrometer Tracing and Zipkin**

## **Objective**
Enable distributed tracing across multiple micro‑services using **Micrometer Tracing with Brave**, the replacement for Spring Cloud Sleuth in Spring Boot 3.4.5. You’ll configure two micro‑services (`UserService` and `OrderService`) to propagate trace IDs and spans automatically and visualize them using **Zipkin**.

---

## **Prerequisites**

| Tool | Install Command (Windows PowerShell) | Verify |
|------|--------------------------------------|--------|
| **Java 17+ JDK** | `winget install --id EclipseAdoptium.Temurin.17.JDK` | `java -version` → `17.*` |
| **Maven 3.9+** | `winget install --id Apache.Maven` | `mvn -v` |
| **Git** | `winget install --id Git.Git` | `git --version` |
| **Docker Desktop** | <https://www.docker.com/products/docker-desktop/> | `docker --version` |
| **IDE** (IntelliJ IDEA / VS Code) | Download → install | — |

> **Tech Explainer — Why these tools?**  
> *Java* runs Spring apps; *Maven* builds them; *Git* backs up and shares your code; *Docker* runs Zipkin in a single command; an *IDE* lets you edit Java comfortably.

---

### **GitHub Setup (one‑time, optional but recommended)**

1. **Create an account.** Go to <https://github.com/> → **Sign up** → follow prompts.  
2. **Create a repository.** After login: **➕ New → New repository** →  
   *Name*: `spring-tracing-labs` → **☑ Add a README** → **Create repository**  
3. **Clone the repo locally.**
   ```powershell
   git clone https://github.com/<your‑username>/spring-tracing-labs.git
   # Expected:
   # Cloning into 'spring-tracing-labs'...
   # remote: Enumerating objects: ...
   ```
4. **Push any project folder during the lab.**
   ```powershell
   git add .
   git commit -m "Add UserService tracing project"
   git push
   # Expected last line:
   # main -> main
   ```

---

## **Part 1: Create and Configure UserService**

> **Tech Explainer — Micrometer Tracing & Brave**  
> *Micrometer Tracing* is the new façade for distributed tracing in Spring. *Brave* is the underlying tracer that sends data to Zipkin. They automatically add `traceId` and `spanId` to logs and HTTP headers.

### **Step 1: Generate Project**
1. Go to <https://start.spring.io>  
2. Fill in: **Group** `com.microservices`, **Artifact** `user-service`, **Spring Boot** `3.4.5`.  
3. Add **Dependencies**: *Spring Web*, *Spring Boot Actuator*.  
4. Click **Generate** and unzip into `spring-tracing-labs/user-service`.

### **Step 2: Import into IDE**
Open IntelliJ/VS Code → **Open Folder** → select `user-service`.

### **Step 3: Add Tracing Dependencies**
*File*: `user-service/pom.xml` (inside `<dependencies>`)
```xml
<!-- Micrometer Tracing bridge -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>

<!-- Zipkin reporter -->
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

### **Step 4: Create REST Controller**
*File*: `user-service/src/main/java/com/microservices/userservice/UserController.java`
```java
package com.microservices.userservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    @GetMapping("/users")
    public String getUsers() {
        return "List of users from UserService";
    }
}
```

### **Step 5: Configure Application Properties**
*File*: `user-service/src/main/resources/application.properties`
```properties
server.port=8081
spring.application.name=user-service

# Enable 100 % sampling so every request is traced
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans

# Expose actuator endpoints
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

# Show trace IDs in logs
logging.pattern.level=%5p [${spring.application.name:},traceId=%X{traceId},spanId=%X{spanId}]
```

### **Step 6: Run UserService**
```powershell
cd user-service
mvn spring-boot:run
# Expected (last lines):
# Started UserServiceApplication in 3.8 seconds
# Tomcat started on port(s): 8081 (http) with context path ''
```

### **Step 7: Test Endpoint**
```powershell
curl http://localhost:8081/users
# Expected:
# List of users from UserService
```
Check the terminal log — you should see `traceId=` and `spanId=` values.

---

## **Part 2: Create and Configure OrderService**

> **Tech Explainer — WebClient**  
> *WebClient* is Spring’s non‑blocking HTTP client. We’ll use it to call `UserService`, so traces propagate automatically via HTTP headers.

### **Step 8: Generate Project**
Repeat Step 1 but with **Artifact** `order-service` and add dependencies: *Spring Web*, *Spring Boot Actuator*, *Spring Reactive Web*.

Unzip into `spring-tracing-labs/order-service`.

### **Step 9: Import into IDE**
Open the `order-service` folder.

### **Step 10: Add Tracing Dependencies**
Add the same two tracing dependencies as `UserService` plus:

```xml
<!-- Optional helper for manual observations -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-observation</artifactId>
</dependency>
```

### **Step 11: Create Controller**
*File*: `order-service/src/main/java/com/microservices/orderservice/OrderController.java`
```java
package com.microservices.orderservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.function.client.WebClient;

@RestController
public class OrderController {

    @Autowired
    private WebClient.Builder webClientBuilder;

    @GetMapping("/orders")
    public String getOrders() {
        String users = webClientBuilder.build()
            .get()
            .uri("http://localhost:8081/users")
            .retrieve()
            .bodyToMono(String.class)
            .block();

        return "Orders from OrderService and Users: " + users;
    }
}
```

### **Step 12: Add WebClient Bean**
Edit `order-service/src/main/java/com/microservices/orderservice/OrderServiceApplication.java`
```java
@Bean
public WebClient.Builder webClientBuilder() {
    return WebClient.builder();
}
```

### **Step 13: Configure Application Properties**
*File*: `order-service/src/main/resources/application.properties`
```properties
server.port=8082
spring.application.name=order-service

management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

logging.pattern.level=%5p [${spring.application.name:},traceId=%X{traceId},spanId=%X{spanId}]
```

### **Step 14: Run OrderService**
```powershell
cd order-service
mvn spring-boot:run
# Expected (last lines):
# Started OrderServiceApplication in 4.0 seconds
# Tomcat started on port(s): 8082 (http)
```

### **Step 15: Test Endpoint**
```powershell
curl http://localhost:8082/orders
# Expected:
# Orders from OrderService and Users: List of users from UserService
```
Look at logs in **both** services — the same `traceId` appears, proving propagation.

---

## **Part 3: Run and View Zipkin UI**

> **Tech Explainer — Zipkin**  
> *Zipkin* is an open‑source tracing system that stores and visualizes spans. We’ll run it in Docker so no local install is needed.

### **Step 16: Start Zipkin using Docker**
```powershell
docker run -d -p 9411:9411 openzipkin/zipkin
# Expected:
# Unable to find image 'openzipkin/zipkin:latest' locally
# latest: Pulling from openzipkin/zipkin
# ...
# Digest: sha256:...
# Status: Downloaded newer image ...
# <container-id>
```

### **Step 17: Access Zipkin UI**
Open a browser:
```
http://localhost:9411
```

### **Step 18: Trigger Trace and View**
1. In a new terminal:
   ```powershell
   curl http://localhost:8082/orders
   ```
2. In Zipkin, click **Run Query** (default service = `order-service`).  
   You should see a trace graph **OrderService → UserService**.

---

## ✅ **Conclusion**
You now have full distributed tracing using **Micrometer Tracing (Brave)** with visualisation in **Zipkin**. Push both projects (`user-service`, `order-service`) to your GitHub repo to showcase end‑to‑end observability skills!

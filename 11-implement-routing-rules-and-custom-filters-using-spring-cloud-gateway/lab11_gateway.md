
# **Lab 11: Implement Routing Rules and Custom Filters Using Spring Cloud Gateway (Spring Boot 3.4.5)**

## **Objective**
Learn how to configure **Spring Cloud Gateway** (on **Spring Boot 3.4.5**) to route requests to various micro‑services. Implement custom filters to intercept requests/responses, utilize route predicates for dynamic routing, and monitor the gateway with actuator endpoints.

---

## **Prerequisite Setup**

| Tool | Install Command (Windows PowerShell) | Verify |
|------|--------------------------------------|--------|
| **Java 17+ JDK** | `winget install --id EclipseAdoptium.Temurin.17.JDK` | `java -version` → `17.*` |
| **Maven 3.9+** | `winget install --id Apache.Maven` | `mvn -v` |
| **Git** | `winget install --id Git.Git` | `git --version` |
| **Docker Desktop** (optional, for containerised Kafka later) | <https://www.docker.com/products/docker-desktop/> | `docker --version` |

> **Tech Explainer — Why we need these tools**  
> *Java 17* runs all Spring projects. *Maven* builds/executes them. *Git* stores code in the cloud. *Docker* lets us run infrastructure components such as Kafka in containers.

---

### **GitHub Setup (one‑time)**

1. **Create an account.** Go to <https://github.com/> → **Sign up** → follow prompts.  
2. **Create a repository.** After login: **➕ New → New repository** →  
   * **Name**: `spring-gateway-labs`  
   * **☑ Add a README** → **Create repository**  
3. **Clone the repo locally.**
   ```powershell
   git clone https://github.com/<your‑username>/spring-gateway-labs.git
   # Expected:
   # Cloning into 'spring-gateway-labs'...
   # remote: Enumerating objects: ...
   ```
4. **Push code during the lab.** Inside each project folder:
   ```powershell
   git add .
   git commit -m "Add ApiGateway project"
   git push
   # Expected last line:
   # main -> main
   ```

---

## **Lab Steps**

### **Part 1: Setting Up the API Gateway**

1. **Generate a new Spring Boot project for `ApiGateway`.**  
   - Go to <https://start.spring.io/>  
   - Use: Group `com.microservices`, Artifact `api-gateway`, Boot `3.4.5`.  
   - Add **Dependencies**: *Spring Cloud Gateway*, *Spring Boot Actuator*.  
   - Click **Generate** → unzip into `spring-gateway-labs/api-gateway`.

   > **Tech Explainer — Spring Cloud Gateway**  
   > A lightweight, reactive API gateway that routes requests, applies filters, and off‑loads cross‑cutting concerns (security, logging). It replaces older Netflix Zuul.

2. **Import the project into your IDE** (IntelliJ IDEA / VS Code). Open the `api-gateway` folder.

3. **Update Maven dependencies**  
   *File*: `api-gateway/pom.xml`  
   Replace the entire `<dependencies>` block with:

   ```xml
   <dependencies>
       <!-- ✅ Spring Cloud Gateway (Reactive) -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-gateway</artifactId>
       </dependency>

       <!-- ✅ WebFlux runtime -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-webflux</artifactId>
       </dependency>

       <!-- Actuator endpoints -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>

       <!-- Tests -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

4. **Create the main application class.**  
   *File*: `api-gateway/src/main/java/com/microservices/apigateway/ApiGatewayApplication.java`

   ```java
   package com.microservices.apigateway;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;

   @SpringBootApplication
   public class ApiGatewayApplication {
       public static void main(String[] args) {
           SpringApplication.run(ApiGatewayApplication.class, args);
       }
   }
   ```

5. **Add route configuration.**  
   *File*: `api-gateway/src/main/resources/application.properties`

   ```properties
   # ----------------------------------------------------------------------
   # Basic service settings
   # ----------------------------------------------------------------------
   spring.application.name=api-gateway
   server.port=8080

   # ----------------------------------------------------------------------
   # Route 1 – user-service
   # ----------------------------------------------------------------------
   spring.cloud.gateway.routes[0].id=user-service
   spring.cloud.gateway.routes[0].uri=http://localhost:8081
   spring.cloud.gateway.routes[0].predicates[0]=Path=/users/**
   spring.cloud.gateway.routes[0].predicates[1]=Query=username
   spring.cloud.gateway.routes[0].filters[0]=AddRequestHeader=X-User-Header, UserServiceHeader

   # ----------------------------------------------------------------------
   # Route 2 – product-service
   # ----------------------------------------------------------------------
   spring.cloud.gateway.routes[1].id=product-service
   spring.cloud.gateway.routes[1].uri=http://localhost:8082
   spring.cloud.gateway.routes[1].predicates[0]=Path=/products/**
   spring.cloud.gateway.routes[1].filters[0]=AddResponseHeader=X-Product-Header, ProductServiceHeader

   # ----------------------------------------------------------------------
   # Actuator / management endpoints
   # ----------------------------------------------------------------------
   management.endpoints.web.exposure.include=*
   management.endpoint.gateway.enabled=true
   management.endpoints.web.base-path=/actuator
   ```

6. **Run the `ApiGateway` application.**
   ```powershell
   cd api-gateway
   mvn spring-boot:run
   # Expected (last lines):
   # Started ApiGatewayApplication in 4.123 s (JVM running for 4.8)
   # Netty started on port(s): 8080
   ```

---

### **Part 2: Setting Up Micro‑services**

7. **Create `UserService`.**  
   - Generate another Spring Boot project (`user-service`) with **Spring Web** dependency.  
   - Unzip into `spring-gateway-labs/user-service`.  
   - *File*: `user-service/src/main/resources/application.properties`
     ```properties
     server.port=8081
     ```
   - *File*: `user-service/src/main/java/com/microservices/userservice/UserController.java`
     ```java
     package com.microservices.userservice;

     import org.springframework.web.bind.annotation.*;

     @RestController
     public class UserController {
         @GetMapping("/users")
         public String getUsers(@RequestParam String username,
                                @RequestHeader(value = "X-User-Header", required = false) String userHeader) {
             System.out.println("Received X-User-Header: " + userHeader);
             return "Users fetched by: " + username + " | Header: " + userHeader;
         }
     }
     ```

8. **Create `ProductService`.**  
   - Generate Spring Boot project (`product-service`) with **Spring Web**.  
   - Unzip into `spring-gateway-labs/product-service`.  
   - *File*: `product-service/src/main/resources/application.properties`
     ```properties
     server.port=8082
     ```
   - *File*: `product-service/src/main/java/com/microservices/productservice/ProductController.java`
     ```java
     package com.microservices.productservice;

     import org.springframework.web.bind.annotation.*;

     @RestController
     public class ProductController {
         @GetMapping("/products")
         public String getProducts() {
             return "List of products from ProductService";
         }
     }
     ```

9. **Run both services in separate terminals.**
   ```powershell
   # Terminal 1
   cd user-service
   mvn spring-boot:run
   # Expected last line: Started UserServiceApplication ... Tomcat started on port(s): 8081

   # Terminal 2
   cd product-service
   mvn spring-boot:run
   # Expected last line: Started ProductServiceApplication ... Tomcat started on port(s): 8082
   ```

---

### **Part 3: Creating and Testing Filters**

10. **Create a global logging filter.**  
    *File*: `api-gateway/src/main/java/com/microservices/apigateway/filter/LoggingFilter.java`
    ```java
    package com.microservices.apigateway.filter;

    import org.slf4j.*;
    import org.springframework.core.annotation.Order;
    import org.springframework.stereotype.Component;
    import org.springframework.cloud.gateway.filter.*;
    import org.springframework.web.server.ServerWebExchange;
    import reactor.core.publisher.Mono;

    @Component
    @Order(1)
    public class LoggingFilter implements GlobalFilter {
        private static final Logger logger = LoggerFactory.getLogger(LoggingFilter.class);

        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            logger.info("Incoming request: " + exchange.getRequest().getURI());
            return chain.filter(exchange).then(Mono.fromRunnable(() ->
                logger.info("Outgoing response: " + exchange.getResponse().getStatusCode())));
        }
    }
    ```

11. **Restart the gateway** (stop & rerun Step 6) and watch the logs.

12. **Test routing through the gateway.**
    ```powershell
    curl "http://localhost:8080/users?username=admin"
    # Expected:
    # Users fetched by: admin | Header: UserServiceHeader

    curl "http://localhost:8080/products"
    # Expected:
    # List of products from ProductService
    # (inspect response headers for X-Product-Header: ProductServiceHeader)
    ```

13. **Verify gateway logs** display “Incoming request” and “Outgoing response”.

---

### **Part 4: Advanced Routing with Predicates**

14. **Test query‑parameter enforcement.**
    ```powershell
    curl "http://localhost:8080/users"
    # Expected: 404 Not Found (because ?username is missing)
    ```

15. **(Optional extension to explore on your own)** Add more predicates or custom filters once the core lab works.

---

### **Part 5: Monitoring the Gateway with Actuator**

16. **Access actuator endpoints.**  
    ```powershell
    curl http://localhost:8080/actuator/gateway/routes
    # Expected: JSON array with `user-service`, `product-service`

    curl http://localhost:8080/actuator/gateway/globalfilters
    # Expected: includes LoggingFilter

    curl http://localhost:8080/actuator/gateway/routefilters
    # Expected: shows AddRequestHeader and AddResponseHeader filters
    ```

---

## ✅ Conclusion
You now have:

- A fully working **API Gateway**  
- Global filters and header manipulation  
- Predicate‑based routing  
- Actuator‑powered visibility  

Push your three projects to GitHub (`spring-gateway-labs`) to showcase your work!

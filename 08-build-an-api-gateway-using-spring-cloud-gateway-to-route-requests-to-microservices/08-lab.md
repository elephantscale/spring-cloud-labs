# **Lab 8: Build an API Gateway Using Spring Cloud Gateway to Route Requests to Microservices**

## **Objective**
Learn how to build an API Gateway using Spring Cloud Gateway. Route requests to multiple microservices, configure custom filters, and enable monitoring.

---

## **Lab Steps**

### **Part 1: Setting Up the API Gateway**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `api-gateway`
     - **Name**: `ApiGateway`
     - **Dependencies**:
       - Spring Cloud Gateway
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the zip file into a folder named `ApiGateway`.

2. **Import the project into your IDE.**

3. **Enable Gateway functionality in `ApiGatewayApplication`.**
   - Open `ApiGatewayApplication.java` in `src/main/java/com/microservices/apigateway`:
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

4. **Configure basic routing in `application.properties`.**
   - Create `application.properties` in `src/main/resources` and add:
     ```properties
     server.port=8080

     spring.cloud.gateway.routes[0].id=user-service
     spring.cloud.gateway.routes[0].uri=http://localhost:8081
     spring.cloud.gateway.routes[0].predicates[0]=Path=/users/**

     spring.cloud.gateway.routes[1].id=product-service
     spring.cloud.gateway.routes[1].uri=http://localhost:8082
     spring.cloud.gateway.routes[1].predicates[0]=Path=/products/**
     ```

5. **Run the API Gateway application.**
   - Start the application:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test basic routing.**
   - Start `UserService` (port `8081`) and `ProductService` (port `8082`).
   - Test routes:
     - `http://localhost:8080/users` routes to `UserService`.
     - `http://localhost:8080/products` routes to `ProductService`.

---

### **Part 2: Setting Up Microservices**

7. **Create `UserService`.**
   - Generate a Spring Boot project with:
     - **Artifact Id**: `user-service`
     - **Dependencies**: Spring Web
   - Add `UserController`:
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

8. **Create `ProductService`.**
   - Generate another Spring Boot project with:
     - **Artifact Id**: `product-service`
     - **Dependencies**: Spring Web
   - Add `ProductController`:
     ```java
     package com.microservices.productservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class ProductController {

         @GetMapping("/products")
         public String getProducts() {
             return "List of products from ProductService";
         }
     }
     ```

9. **Run `UserService` and `ProductService`.**
   - Set ports in their `application.properties` files:
     ```properties
     server.port=8081
     ```
     ```properties
     server.port=8082
     ```
   - Verify endpoints:
     - `http://localhost:8081/users`
     - `http://localhost:8082/products`

---

### **Part 3: Adding Global Filters**

10. **Create a global logging filter.**
    - Add `LoggingFilter.java` in `src/main/java/com/microservices/apigateway`:
      ```java
      package com.microservices.apigateway;

      import org.slf4j.Logger;
      import org.slf4j.LoggerFactory;
      import org.springframework.cloud.gateway.filter.GlobalFilter;
      import org.springframework.core.annotation.Order;
      import org.springframework.stereotype.Component;
      import reactor.core.publisher.Mono;

      @Component
      @Order(1)
      public class LoggingFilter implements GlobalFilter {

          private static final Logger logger = LoggerFactory.getLogger(LoggingFilter.class);

          @Override
          public Mono<Void> filter(org.springframework.web.server.ServerWebExchange exchange,
                                   org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {
              logger.info("Incoming request: " + exchange.getRequest().getURI());
              return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                  logger.info("Outgoing response: " + exchange.getResponse().getStatusCode());
              }));
          }
      }
      ```

11. **Test the global logging filter.**
    - Access `/users` or `/products` and verify logs in the console.

---

### **Part 4: Customizing Routes**

12. **Add a route with custom headers.**
    - Update `application.properties`:
      ```properties
      spring.cloud.gateway.routes[2].id=custom-route
      spring.cloud.gateway.routes[2].uri=http://httpbin.org:80
      spring.cloud.gateway.routes[2].predicates[0]=Path=/custom/**
      spring.cloud.gateway.routes[2].filters[0]=AddRequestHeader=X-Custom-Header, CustomValue
      ```

13. **Test the custom route.**
    - Access `http://localhost:8080/custom` and verify the header is added.

14. **Add a rate limiter.**
    - Add the `spring-boot-starter-data-redis` dependency in `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
      ```
    - Configure rate limiting in `application.properties`:
      ```properties
      spring.cloud.gateway.routes[3].id=rate-limited-route
      spring.cloud.gateway.routes[3].uri=http://httpbin.org:80
      spring.cloud.gateway.routes[3].predicates[0]=Path=/rate-limited/**
      spring.cloud.gateway.routes[3].filters[0]=RequestRateLimiter=redis-rate-limiter.replenishRate=2, redis-rate-limiter.burstCapacity=2
      ```

15. **Test the rate limiter.**
    - Access `/rate-limited` multiple times and verify only 2 requests are allowed per burst.

---

### **Part 5: Monitoring**

16. **Enable Spring Boot Actuator.**
    - Ensure this dependency is added:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose management endpoints.**
    - Add to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=routes,filters
      ```

18. **View active routes and filters.**
    - Access:
      - `http://localhost:8080/actuator/routes`
      - `http://localhost:8080/actuator/filters`

---

### **Optional Exercises**

1. **Add a custom filter for response headers.**
   - Modify outgoing responses to include a custom header.

2. **Test load balancing.**
   - Deploy multiple instances of `UserService` and test gateway load balancing.

3. **Integrate Circuit Breakers.**
   - Add Resilience4j-based circuit breakers to specific routes.

---

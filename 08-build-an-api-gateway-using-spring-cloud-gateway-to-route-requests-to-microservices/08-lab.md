# **Lab 8: Build an API Gateway Using Spring Cloud Gateway to Route Requests to Microservices (Spring Boot 3.4.1)**

## **Objective**
Learn how to build an **API Gateway** using **Spring Cloud Gateway** (with Spring Boot **3.4.1**). You will configure routes to multiple microservices, add global/custom filters, and enable monitoring endpoints for deeper insights.

---

## **Lab Steps**

### **Part 1: Setting Up the API Gateway**

1. **Generate a new Spring Boot project for `ApiGateway`.**
   - Go to [https://start.spring.io/](https://start.spring.io/).
   - **Spring Boot Version**: **3.4.1**
   - **Group Id**: `com.microservices`
   - **Artifact Id**: `api-gateway`
   - **Name**: `ApiGateway`
   - **Dependencies**:
     - Spring Cloud Gateway
     - Spring Boot Actuator
   - Extract into a folder named `ApiGateway`.

2. **Import the project into your IDE.**

3. **Enable Gateway functionality.**
   - In `ApiGatewayApplication.java` (e.g., `src/main/java/com/microservices/apigateway`):
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
   - In `src/main/resources/application.properties`:
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
   - From `ApiGateway` folder:
     ```bash
     ./mvnw spring-boot:run
     ```
   - The gateway starts on port **8080**.

6. **Test basic routing.**
   - Make sure you have two microservices: `UserService` on **8081** and `ProductService` on **8082** running.
   - Confirm:
     - `http://localhost:8080/users` routes to `UserService`
     - `http://localhost:8080/products` routes to `ProductService`

---

### **Part 2: Setting Up Microservices**

7. **Create or confirm `UserService`.**
   - Spring Boot app with **Spring Web**. Suppose `application.properties` has:
     ```properties
     server.port=8081
     ```
   - A simple controller:
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

8. **Create or confirm `ProductService`.**
   - Another Spring Boot app with **Spring Web**. `application.properties`:
     ```properties
     server.port=8082
     ```
   - A simple controller:
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

9. **Ensure both microservices run** at ports **8081** and **8082**, respectively.

---

### **Part 3: Adding Global Filters**

10. **Create a global logging filter.**
    - In `src/main/java/com/microservices/apigateway/LoggingFilter.java`:
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
              logger.info("Incoming request: {}", exchange.getRequest().getURI());
              return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                  logger.info("Outgoing response: {}", exchange.getResponse().getStatusCode());
              }));
          }
      }
      ```
    - This logs requests and responses at a **global** level.

11. **Test the global logging filter.**
    - Hit `/users` or `/products` via the gateway.
    - Observe logs in the console. You should see `Incoming request` and `Outgoing response`.

---

### **Part 4: Customizing Routes**

12. **Add a route with custom headers.**
    - In `application.properties`, add a third route:
      ```properties
      spring.cloud.gateway.routes[2].id=custom-route
      spring.cloud.gateway.routes[2].uri=http://httpbin.org:80
      spring.cloud.gateway.routes[2].predicates[0]=Path=/custom/**
      spring.cloud.gateway.routes[2].filters[0]=AddRequestHeader=X-Custom-Header, CustomValue
      ```
    - This route sends any request matching `/custom/**` to [httpbin.org](http://httpbin.org) with an extra header.

13. **Test the custom route.**
    - Access:
      ```
      http://localhost:8080/custom
      ```
    - You should see a response from httpbin, and in the logs or the httpbin response, the `X-Custom-Header` is attached.

14. **Add a rate limiter.**
    - **Add Redis dependency** in `pom.xml` if you want advanced rate-limiting:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
      ```
    - Then in `application.properties`, add:
      ```properties
      spring.cloud.gateway.routes[3].id=rate-limited-route
      spring.cloud.gateway.routes[3].uri=http://httpbin.org:80
      spring.cloud.gateway.routes[3].predicates[0]=Path=/rate-limited/**
      spring.cloud.gateway.routes[3].filters[0]=RequestRateLimiter=redis-rate-limiter.replenishRate=2, redis-rate-limiter.burstCapacity=2
      ```
    - This allows only **2** requests in a certain timeframe before rate-limiting further calls.

15. **Test the rate limiter.**
    - Access `/rate-limited` multiple times in quick succession.
    - After the threshold, you should see errors or rate-limited responses.

---

### **Part 5: Monitoring**

16. **Enable Spring Boot Actuator.**
    - Ensure `spring-boot-starter-actuator` is in `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose management endpoints.**
    - In `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=routes,filters
      ```

18. **View active routes and filters.**
    - Check:
      - `http://localhost:8080/actuator/routes`
      - `http://localhost:8080/actuator/filters`
    - You’ll see your configured routes and filters listed.

---

## **Optional Exercises**

1. **Add a custom filter for response headers.**
   - Create a GatewayFilterFactory to manipulate outbound responses.

2. **Test load balancing.**
   - Run multiple instances of `UserService` on different ports, e.g. `8081` and `8083`.
   - Adjust your route to a **LoadBalanced** approach (like `uri: lb://user-service`) if using Discovery or custom filters.

3. **Integrate Circuit Breakers.**
   - Use **Resilience4j** on specific routes to handle downstream failures gracefully.

---

## **Conclusion**
With this lab, you:
- **Created an API Gateway** with **Spring Cloud Gateway** on **Spring Boot 3.4.1**.
- **Configured routes** for multiple microservices (`UserService`, `ProductService`).
- **Implemented custom filters** (logging, custom headers, rate limiting).
- **Monitored** Gateway routes and filters via Actuator endpoints.

Your gateway now provides a single entry point for your microservices, enabling consistent cross-cutting behaviors (logging, security, rate limiting, etc.) in one place.

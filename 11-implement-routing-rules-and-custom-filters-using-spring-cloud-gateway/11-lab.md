# **Lab 11: Implement Routing Rules and Custom Filters Using Spring Cloud Gateway (Spring Boot 3.4.1)**

## **Objective**
Learn how to configure **Spring Cloud Gateway** (on **Spring Boot 3.4.1**) to route requests to various microservices. Implement custom filters to intercept requests/responses, utilize route predicates for dynamic routing, and monitor the gateway with actuator endpoints.

---

## **Lab Steps**

### **Part 1: Setting Up the API Gateway**

1. **Generate a new Spring Boot project for `ApiGateway`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `api-gateway`
     - **Name**: `ApiGateway`
     - **Dependencies**:
       - Spring Cloud Gateway
       - Spring Boot Actuator
   - Extract into `ApiGateway`.

2. **Import the project into your IDE.**

3. **Enable Spring Cloud Gateway.**
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

4. **Configure default routes in `application.properties`.**
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

5. **Run the `ApiGateway` application.**
   - From the `ApiGateway` folder:
     ```bash
     ./mvnw spring-boot:run
     ```
   - Gateway listens on **port 8080**.

6. **Test basic routing.**
   - Have two microservices: `UserService` (on **8081**) and `ProductService` (on **8082**) running.
   - Access:
     - `http://localhost:8080/users` → Routes to `UserService`
     - `http://localhost:8080/products` → Routes to `ProductService`

---

### **Part 2: Setting Up Microservices**

7. **Create `UserService`.**
   - Another Spring Boot project with **Spring Web**:
     ```properties
     # application.properties
     server.port=8081
     ```
   - `UserController.java`:
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
   - Another Spring Boot project:
     ```properties
     # application.properties
     server.port=8082
     ```
   - `ProductController.java`:
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

9. **Ensure both services run** on **8081** and **8082**.

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
    - Logs each incoming request URI and outgoing response status.

11. **Test the global logging filter.**
    - Hit `http://localhost:8080/users` or `/products`.
    - Check logs for messages like `Incoming request` and `Outgoing response`.

---

### **Part 4: Implementing Route-Specific Filters**

12. **Add a request header filter.**
    - In `application.properties`, for the user-service route:
      ```properties
      spring.cloud.gateway.routes[0].filters[0]=AddRequestHeader=X-User-Header, UserServiceHeader
      ```
    - This adds an `X-User-Header` to all requests passing via `/users/**`.

13. **Verify the request header filter.**
    - Use a client (Postman, curl) to GET `http://localhost:8080/users`.
    - Check the request logs (on the user-service side or using a proxy) for `X-User-Header: UserServiceHeader`.

14. **Add a custom response modification filter.**
    - Create `CustomResponseFilter.java`:
      ```java
      package com.microservices.apigateway;

      import org.springframework.cloud.gateway.filter.GatewayFilter;
      import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
      import org.springframework.stereotype.Component;
      import reactor.core.publisher.Mono;

      @Component
      public class CustomResponseFilter extends AbstractGatewayFilterFactory<Object> {

          @Override
          public GatewayFilter apply(Object config) {
              return (exchange, chain) -> chain.filter(exchange).then(Mono.fromRunnable(() -> {
                  exchange.getResponse().getHeaders().add("X-Response-Header", "ModifiedResponse");
              }));
          }
      }
      ```
    - This filter appends an `X-Response-Header` to outgoing responses.

15. **Apply the custom response filter to product-service route.**
    - In `application.properties`:
      ```properties
      spring.cloud.gateway.routes[1].filters[0]=CustomResponseFilter
      ```
    - When requests match the `/products/**` route, the `X-Response-Header` is added.

16. **Test the response modification filter.**
    - Access `http://localhost:8080/products`.
    - Check the response headers for `X-Response-Header: ModifiedResponse`.

---

### **Part 5: Adding Predicate Rules**

17. **Add a query parameter predicate.**
    - For user-service, in `application.properties`:
      ```properties
      spring.cloud.gateway.routes[0].predicates[1]=Query=username
      ```
    - This route only matches if `?username=...` is present in the query string.

18. **Test the query parameter predicate.**
    - Access `http://localhost:8080/users?username=test`.
    - Without the `username` param, the route might not match.

19. **Add a time-based route predicate.**
    - For product-service, in `application.properties`:
      ```properties
      spring.cloud.gateway.routes[1].predicates[1]=After=2024-01-01T00:00:00Z
      ```
    - This route only becomes valid after January 1, 2024.

20. **Test the time-based predicate.**
    - If the current date is before 2024-01-01, `/products/**` might return `404` or no route match.

---

### **Part 6: Monitoring and Management**

21. **Enable Spring Boot Actuator.**
    - In `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

22. **Expose management endpoints.**
    - In `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=routes,filters
      ```
    - This allows you to view gateway routes and filters via Actuator.

23. **View active routes.**
    - Visit:
      ```
      http://localhost:8080/actuator/routes
      ```
    - See all configured routes.

24. **View available filters.**
    - Visit:
      ```
      http://localhost:8080/actuator/filters
      ```
    - Lists all filter classes (including custom ones).

---

## **Optional Exercises**

1. **Create a custom rate-limiting filter.**
   - Implement a GatewayFilterFactory or use Redis-based `RequestRateLimiter` for specific routes.

2. **Integrate Circuit Breakers.**
   - Combine with **Resilience4j** (or Spring Cloud CircuitBreaker) to handle downstream service failures.

3. **Load Balancing.**
   - If using **Eureka** or another discovery system, set `uri=lb://user-service` to load-balance multiple instances.

---

## **Conclusion**
By completing this lab:
- **Configured routing rules** in a **Spring Cloud Gateway** (Spring Boot 3.4.1) application.
- **Implemented custom filters** globally and route-specific to manipulate requests/responses.
- **Used predicates** (query parameters, time-based) to dynamically match routes.
- **Monitored** the gateway via Spring Boot Actuator endpoints.

You now have an **API Gateway** centralizing cross-cutting concerns like logging, header manipulation, rate limiting, time-based routes, and more for your microservice environment.

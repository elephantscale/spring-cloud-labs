# **Lab 8: Build an API Gateway Using Spring Cloud Gateway to Route Requests to Microservices**

## **Objective**
Learn how to build an API Gateway using Spring Cloud Gateway. Route requests to multiple microservices and configure basic filters for request manipulation.

---

## **Lab Steps**

### **Part 1: Setting Up the API Gateway**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `api-gateway`
     - **Name**: `ApiGateway`
     - **Dependencies**:
       - Spring Cloud Gateway
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `ApiGateway`.

2. **Import the project into your IDE.**
   - Open your IDE and import the `ApiGateway` project as a Maven project.

3. **Enable Gateway functionality in `ApiGatewayApplication`.**
   - Open the `ApiGatewayApplication.java` file in the `src/main/java/com/microservices/apigateway` folder.
   - Add the `@SpringBootApplication` annotation:
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
   - Open the `src/main/resources/application.properties` file and add:
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
   - Start the `ApiGateway` application using:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test basic routing.**
   - Start `UserService` (on port `8081`) and `ProductService` (on port `8082`).
   - Use Postman or a browser to test routing:
     - Access `http://localhost:8080/users` to verify it routes to `UserService`.
     - Access `http://localhost:8080/products` to verify it routes to `ProductService`.

---

### **Part 2: Setting Up Microservices**

7. **Create `UserService`.**
   - Generate a new Spring Boot project using Spring Initializr with the following configuration:
     - **Artifact Id**: `user-service`
     - **Dependencies**:
       - Spring Web
     - Add a sample REST endpoint in `UserController`:
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
   - Generate another Spring Boot project using Spring Initializr with the following configuration:
     - **Artifact Id**: `product-service`
     - **Dependencies**:
       - Spring Web
     - Add a sample REST endpoint in `ProductController`:
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
   - Start both services on ports `8081` and `8082`, respectively.
   - Verify their endpoints by accessing:
     - `http://localhost:8081/users`
     - `http://localhost:8082/products`

---

### **Part 3: Adding Global Filters to the Gateway**

10. **Add a global filter for logging.**
    - Create a new file `GlobalFilter.java` in the `src/main/java/com/microservices/apigateway` folder:
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
          public org.springframework.web.server.ServerWebExchange filter(
              org.springframework.web.server.ServerWebExchange exchange,
              org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {

              logger.info("Incoming request: " + exchange.getRequest().getURI());
              return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                  logger.info("Outgoing response: " + exchange.getResponse().getStatusCode());
              }));
          }
      }
      ```

11. **Test the global logging filter.**
    - Access any route (e.g., `/users` or `/products`) and verify logs are generated for requests and responses.

---

### **Part 4: Applying Predicate and Filter Customizations**

12. **Add a route with custom predicates and filters.**
    - Update `application.properties`:
      ```properties
      spring.cloud.gateway.routes[2].id=custom-route
      spring.cloud.gateway.routes[2].uri=http://httpbin.org:80
      spring.cloud.gateway.routes[2].predicates[0]=Path=/custom/**
      spring.cloud.gateway.routes[2].filters[0]=AddRequestHeader=X-Custom-Header, CustomValue
      ```

13. **Test the custom route.**
    - Access `http://localhost:8080/custom` and verify that the custom header is added.

14. **Add a rate limiter filter.**
    - Add the `spring-boot-starter-data-redis` dependency to `pom.xml`:
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
      spring.cloud.gateway.routes[3].filters[0]=RequestRateLimiter=redis-rate-limiter
      ```

15. **Test the rate limiter.**
    - Access `http://localhost:8080/rate-limited` multiple times and verify that the rate limiter restricts access after the allowed number of requests.

---

### **Part 5: Monitoring and Management**

16. **Enable Spring Boot Actuator.**
    - Ensure the following dependency is added in `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose management endpoints.**
    - Add the following to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=routes,filters
      ```

18. **View active routes.**
    - Access the management endpoint:
      ```
      http://localhost:8080/actuator/routes
      ```
    - Verify that all configured routes are displayed.

19. **Monitor filters.**
    - Access the management endpoint:
      ```
      http://localhost:8080/actuator/filters
      ```
    - Verify that the active filters are displayed.

20. **Test load balancing (if applicable).**
    - Deploy multiple instances of `UserService` and verify that the gateway balances requests across them.

---

### **Optional Exercises (20 mins)**

1. **Add a custom filter to modify response headers.**
   - Implement a filter that adds a custom header to all responses.

2. **Integrate Circuit Breakers with routes.**
   - Add Resilience4j-based circuit breakers to specific routes and test their behavior.


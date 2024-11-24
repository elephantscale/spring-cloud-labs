# **Lab 11: Implement Routing Rules and Custom Filters Using Spring Cloud Gateway**

## **Objective**
Learn how to configure routing rules and implement custom filters in Spring Cloud Gateway to handle request manipulation and response processing.

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

3. **Enable Spring Cloud Gateway in `ApiGatewayApplication`.**
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

4. **Configure default routes in `application.properties`.**
   - Add the following routing configuration in `src/main/resources/application.properties`:
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
   - Start the application using:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test the default routes.**
   - Ensure `UserService` (port `8081`) and `ProductService` (port `8082`) are running.
   - Use Postman or a browser to test routing:
     - Access `http://localhost:8080/users` to verify it routes to `UserService`.
     - Access `http://localhost:8080/products` to verify it routes to `ProductService`.

---

### **Part 2: Adding Custom Filters**

7. **Create a custom global filter.**
   - Create a new file `LoggingFilter.java` in `src/main/java/com/microservices/apigateway`:
     ```java
     package com.microservices.apigateway;

     import org.slf4j.Logger;
     import org.slf4j.LoggerFactory;
     import org.springframework.cloud.gateway.filter.GlobalFilter;
     import org.springframework.stereotype.Component;
     import reactor.core.publisher.Mono;

     @Component
     public class LoggingFilter implements GlobalFilter {

         private static final Logger logger = LoggerFactory.getLogger(LoggingFilter.class);

         @Override
         public org.springframework.web.server.ServerWebExchange filter(
             org.springframework.web.server.ServerWebExchange exchange,
             org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {

             logger.info("Request Path: " + exchange.getRequest().getPath());
             return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                 logger.info("Response Status Code: " + exchange.getResponse().getStatusCode());
             }));
         }
     }
     ```

8. **Test the custom filter.**
   - Access `/users` or `/products` endpoints and observe the logs to ensure the filter is logging request paths and response status codes.

---

### **Part 3: Implementing Route-Specific Filters**

9. **Add a request header filter.**
   - Update `application.properties` to include a filter for the `user-service` route:
     ```properties
     spring.cloud.gateway.routes[0].filters[0]=AddRequestHeader=X-User-Header, UserServiceHeader
     ```

10. **Verify the request header filter.**
    - Use Postman to access `http://localhost:8080/users`.
    - Confirm that the `X-User-Header` is included in the request headers.

11. **Add a custom response modification filter.**
    - Create a new filter file `CustomResponseFilter.java`:
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

12. **Apply the custom response filter to a route.**
    - Update `application.properties` for the `product-service` route:
      ```properties
      spring.cloud.gateway.routes[1].filters[1]=CustomResponseFilter
      ```

13. **Test the response modification filter.**
    - Access `http://localhost:8080/products`.
    - Verify that the `X-Response-Header` is included in the response headers.

---

### **Part 4: Adding Predicate Rules**

14. **Add a query parameter predicate.**
    - Add the following to `application.properties` for `user-service`:
      ```properties
      spring.cloud.gateway.routes[0].predicates[1]=Query=username
      ```

15. **Test the query parameter predicate.**
    - Access `http://localhost:8080/users?username=test` to verify that the route matches only when the `username` parameter is included.

16. **Add a time-based route predicate.**
    - Configure a time-based predicate for `product-service`:
      ```properties
      spring.cloud.gateway.routes[1].predicates[1]=After=2024-01-01T00:00:00Z
      ```

17. **Test the time-based predicate.**
    - Access `http://localhost:8080/products` and verify that it only routes after the specified date.

---

### **Part 5: Monitoring and Management**

18. **Enable Spring Boot Actuator.**
    - Ensure the following dependency is added in `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

19. **Expose management endpoints.**
    - Add the following to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=routes,filters
      ```

20. **View active routes.**
    - Access the management endpoint:
      ```
      http://localhost:8080/actuator/routes
      ```
    - Verify that all configured routes are displayed.

21. **View available filters.**
    - Access the management endpoint:
      ```
      http://localhost:8080/actuator/filters
      ```
    - Verify that all active filters are displayed.

---

### **Optional Exercises (20 mins)**

1. **Create a custom rate-limiting filter.**
   - Use Redis to implement and test rate limiting for specific routes.

2. **Integrate Circuit Breakers.**
   - Add Resilience4j circuit breakers to routes and test their behavior under failure conditions.

---
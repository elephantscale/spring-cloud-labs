# **Lab 12: Implement Distributed Tracing with Spring Cloud Sleuth to Track Requests Across Services**

## **Objective**
Learn how to use Spring Cloud Sleuth to enable distributed tracing across microservices. Understand how Sleuth automatically propagates trace IDs and spans to help track requests across services.

---

## **Lab Steps**

### **Part 1: Setting Up Microservices for Tracing**

1. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Sleuth
   - Click **Generate** to download the project zip file.
   - Extract the zip file into a folder named `UserService`.

2. **Import the `UserService` project into your IDE.**

3. **Create a REST controller in `UserService`.**
   - Add the file `UserController.java`:
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

4. **Run the `UserService`.**
   - Start the application:
     ```bash
     ./mvnw spring-boot:run
     ```

5. **Test the `/users` endpoint.**
   - Use Postman or a browser to access:
     ```
     http://localhost:8081/users
     ```

6. **Verify the Sleuth tracing ID is logged.**
   - Check the logs to confirm that Sleuth automatically adds trace IDs and spans to each request.

---

### **Part 2: Setting Up Another Microservice**

7. **Generate a new Spring Boot project for `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Sleuth
       - Spring WebClient
   - Extract the zip file into a folder named `OrderService`.

8. **Import the `OrderService` project into your IDE.**

9. **Create a REST controller in `OrderService`.**
   - Add the file `OrderController.java`:
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

10. **Add WebClient configuration to `OrderService`.**
    - Add the following in `OrderServiceApplication.java`:
      ```java
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.context.annotation.Bean;
      import org.springframework.web.reactive.function.client.WebClient;

      @SpringBootApplication
      public class OrderServiceApplication {

          public static void main(String[] args) {
              SpringApplication.run(OrderServiceApplication.class, args);
          }

          @Bean
          public WebClient.Builder webClientBuilder() {
              return WebClient.builder();
          }
      }
      ```

11. **Run the `OrderService`.**
    ```bash
    ./mvnw spring-boot:run
    ```

12. **Test the `/orders` endpoint.**
    - Use Postman or a browser to access:
      ```
      http://localhost:8082/orders
      ```
    - Verify that the response includes data from both `OrderService` and `UserService`.

13. **Verify distributed tracing in logs.**
    - Check the logs of both `UserService` and `OrderService`.
    - Confirm that the same trace ID is propagated across both services for a single request.

---

### **Part 3: Adding Sleuth Customization**

14. **Customize tracing in `UserService`.**
    - Add the following properties in `application.yml`:
      ```yaml
      spring:
        sleuth:
          sampler:
            probability: 1.0
          trace-id128: true
      ```

15. **Customize tracing in `OrderService`.**
    - Add the same properties as in `UserService` to ensure consistent tracing.

16. **Log trace IDs in responses.**
    - Modify `UserController` in `UserService` to include trace IDs:
      ```java
      import org.slf4j.MDC;

      @GetMapping("/users")
      public String getUsers() {
          String traceId = MDC.get("traceId");
          return "Trace ID: " + traceId + ", Users: [User1, User2]";
      }
      ```

17. **Test the customized tracing.**
    - Call the `/orders` endpoint and verify that trace IDs are returned in responses.

---

### **Part 4: Monitoring Tracing**

18. **Enable Spring Boot Actuator.**
    - Ensure the following dependency is added in both services' `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

19. **Expose tracing-related actuator endpoints.**
    - Add the following to `application.yml` in both services:
      ```yaml
      management:
        endpoints:
          web:
            exposure:
              include: trace,health
      ```

20. **Access tracing actuator endpoints.**
    - Use the following URLs to view trace information:
      - For `UserService`: `http://localhost:8081/actuator/trace`
      - For `OrderService`: `http://localhost:8082/actuator/trace`

---

### **Optional Exercises**

1. **Integrate with Zipkin or Jaeger.**
   - Install and run Zipkin locally or use a hosted instance.
   - Configure Sleuth to export traces to Zipkin:
     ```yaml
     spring:
       zipkin:
         base-url: http://localhost:9411
       sleuth:
         sampler:
           probability: 1.0
     ```
   - Verify traces in the Zipkin UI.

2. **Add more microservices.**
   - Create additional services (e.g., `ProductService`) and verify distributed tracing propagates across all services.

---

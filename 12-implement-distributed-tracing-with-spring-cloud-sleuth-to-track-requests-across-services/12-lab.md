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
   - Extract the downloaded zip file into a folder named `UserService`.

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

4. **Configure `UserService` for tracing.**
   - Add the following properties to `application.properties`:
     ```properties
     server.port=8081

     spring.application.name=user-service
     spring.sleuth.sampler.probability=1.0
     spring.sleuth.trace-id128=true
     ```

5. **Run the `UserService`.**
   - Start the application:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test the `/users` endpoint.**
   - Use Postman or a browser to access:
     ```
     http://localhost:8081/users
     ```

7. **Verify Sleuth tracing in logs.**
   - Check the logs to confirm that Sleuth automatically adds trace IDs and spans to each request.

---

### **Part 2: Setting Up Another Microservice**

8. **Generate a new Spring Boot project for `OrderService`.**
   - Configure the project:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Sleuth
       - Spring WebClient
   - Extract the downloaded zip file into a folder named `OrderService`.

9. **Import the `OrderService` project into your IDE.**

10. **Create a REST controller in `OrderService`.**
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

11. **Configure `OrderService` for tracing.**
    - Add the following properties to `application.properties`:
      ```properties
      server.port=8082

      spring.application.name=order-service
      spring.sleuth.sampler.probability=1.0
      spring.sleuth.trace-id128=true
      ```

12. **Add WebClient configuration to `OrderService`.**
    - Add the following bean in `OrderServiceApplication.java`:
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

13. **Run the `OrderService`.**
    ```bash
    ./mvnw spring-boot:run
    ```

14. **Test the `/orders` endpoint.**
    - Access:
      ```
      http://localhost:8082/orders
      ```
    - Verify that the response includes data from both `OrderService` and `UserService`.

15. **Verify distributed tracing in logs.**
    - Check the logs of both `UserService` and `OrderService`.
    - Confirm that the same trace ID is propagated across both services for a single request.

---

### **Part 3: Monitoring Tracing**

16. **Enable Spring Boot Actuator.**
    - Add the following dependency in both services' `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose tracing-related actuator endpoints.**
    - Add the following properties in `application.properties` in both services:
      ```properties
      management.endpoints.web.exposure.include=trace,health
      ```

18. **Access tracing actuator endpoints.**
    - Use:
      - For `UserService`: `http://localhost:8081/actuator/trace`
      - For `OrderService`: `http://localhost:8082/actuator/trace`

---

### **Optional Exercises**

1. **Integrate with Zipkin or Jaeger.**
   - Install and run Zipkin locally or use a hosted instance.
   - Configure Sleuth to export traces to Zipkin:
     ```properties
     spring.zipkin.base-url=http://localhost:9411
     spring.sleuth.sampler.probability=1.0
     ```
   - Verify traces in the Zipkin UI.

2. **Add more microservices.**
   - Create additional services (e.g., `ProductService`) and verify distributed tracing propagates across all services.

---

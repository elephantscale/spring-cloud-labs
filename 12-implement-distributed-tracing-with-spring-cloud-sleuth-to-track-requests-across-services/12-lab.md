# **Lab 12: Implement Distributed Tracing with Spring Cloud Sleuth to Track Requests Across Services (Spring Boot 3.4.1)**

## **Objective**
Enable **distributed tracing** across multiple microservices using **Spring Cloud Sleuth** (with Spring Boot **3.4.1**). You will configure two services (`UserService` and `OrderService`) that pass **trace IDs** and **spans** automatically, making it easy to track end-to-end requests.

---

## **Lab Steps**

### **Part 1: Setting Up Microservices for Tracing**

#### **UserService**

1. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - **Spring Boot Version**: **3.4.1**
   - **Group Id**: `com.microservices`
   - **Artifact Id**: `user-service`
   - **Dependencies**:
     - Spring Web
     - Spring Boot Actuator
     - Spring Cloud Sleuth
   - Extract into `UserService`.

2. **Import `UserService` into your IDE.**

3. **Add a REST controller.**
   - In `src/main/java/com/microservices/userservice/UserController.java`:
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

4. **Configure tracing in `application.properties`.**
   - In `src/main/resources/application.properties`:
     ```properties
     server.port=8081

     spring.application.name=user-service
     spring.sleuth.sampler.probability=1.0
     spring.sleuth.trace-id128=true
     ```
   - `sampler.probability=1.0` ensures **all** requests are sampled (for demonstration).

5. **Run `UserService`.**
   - From `UserService`:
     ```bash
     ./mvnw spring-boot:run
     ```
   - It starts on **port 8081**.

6. **Test the `/users` endpoint.**
   - Visit:
     ```
     http://localhost:8081/users
     ```
   - Logs should show something like `[traceId=..., spanId=...]`.

---

### **Part 2: Creating Another Microservice for Downstream Calls**

#### **OrderService**

7. **Generate a new project for `OrderService`.**
   - Similar steps:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Sleuth
       - Spring WebClient
   - Extract into `OrderService`.

8. **Import `OrderService`** into your IDE.

9. **Create a REST controller in `OrderService`.**
   - In `src/main/java/com/microservices/orderservice/OrderController.java`:
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

10. **Configure tracing in `application.properties`.**
    - In `src/main/resources/application.properties`:
      ```properties
      server.port=8082

      spring.application.name=order-service
      spring.sleuth.sampler.probability=1.0
      spring.sleuth.trace-id128=true
      ```

11. **Add WebClient configuration.**
    - In `OrderServiceApplication.java`:
      ```java
      package com.microservices.orderservice;

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

12. **Run `OrderService`.**
    - From `OrderService`:
      ```bash
      ./mvnw spring-boot:run
      ```
    - It starts on **port 8082**.

13. **Test the `/orders` endpoint.**
    - Visit:
      ```
      http://localhost:8082/orders
      ```
    - It calls `http://localhost:8081/users`, returning something like **"Orders from OrderService and Users: List of users from UserService"**.

14. **Verify distributed tracing in logs.**
    - Check logs in both `UserService` and `OrderService`.
    - You should see the same **trace ID** appear in both logs for a single request path, indicating correlated traces across services.

---

### **Part 3: Monitoring Tracing**

15. **Enable Spring Boot Actuator for both services.**
    - In each `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```
    - In each `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=trace,health
      ```
    - (Note: As of Spring Boot 3.x, the `/actuator/trace` endpoint is no longer enabled by default, but you can confirm some Sleuth data via logs or other endpoints.)

16. **Access tracing actuator endpoints.**
    - For **UserService**:
      ```
      http://localhost:8081/actuator/trace
      ```
    - For **OrderService**:
      ```
      http://localhost:8082/actuator/trace
      ```
    - Depending on the exact Spring Boot version and configuration, you may see partial or deprecated details. You might rely more on logs for actual trace output.

---

## **Optional Exercises**

1. **Integrate with Zipkin or Jaeger.**
   - Download and run **Zipkin** (on port **9411**).
   - In your `application.properties`:
     ```properties
     spring.zipkin.base-url=http://localhost:9411
     spring.sleuth.sampler.probability=1.0
     ```
   - Check the **Zipkin UI** at `http://localhost:9411` to see distributed traces visually.

2. **Add more microservices.**
   - e.g., `ProductService`, confirm trace IDs propagate across all calls.

3. **Simulate failure or slow calls.**
   - Add timeouts or exceptions in `UserService`. Observe logs to see how the trace records the error or delayed spans.

---

## **Conclusion**
By finishing this lab, you:
- **Enabled distributed tracing** for multiple microservices (UserService, OrderService) using **Spring Cloud Sleuth** on **Spring Boot 3.4.1**.
- **Observed** correlated logs with consistent trace IDs across service boundaries.
- **(Optionally)** integrated **Zipkin/Jaeger** for a visual trace analysis.

You can now easily follow a single request flow across different services, drastically simplifying debugging and performance analysis in your microservice architecture!

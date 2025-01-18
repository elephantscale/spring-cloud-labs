# **Lab 6: Implement Circuit Breakers Using Resilience4j to Manage Service Failures**

## **Objective**
Learn how to implement circuit breakers for fault tolerance using Resilience4j. Configure fallback methods to handle service failures gracefully and monitor service health.

---

## **Lab Steps**

### **Part 1: Setting Up the Base Service**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Resilience4j Spring Boot Starter
   - Click **Generate** to download the project zip file.
   - Extract the zip file into a folder named `UserService`.

2. **Import the project into your IDE.**
   - Import `UserService` as a Maven project.

3. **Add a REST controller to simulate a user endpoint.**
   - Create `UserController.java` in `src/main/java/com/microservices/userservice`:
     ```java
     package com.microservices.userservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class UserController {

         @GetMapping("/users")
         public String getUsers() {
             return "List of users";
         }
     }
     ```

4. **Run the application.**
   - Start the `UserService` application:
     ```bash
     ./mvnw spring-boot:run
     ```

5. **Test the `/users` endpoint.**
   - Access:
     ```
     http://localhost:8080/users
     ```
   - Verify the response: `"List of users"`.

---

### **Part 2: Adding a Circuit Breaker**

6. **Simulate a dependent service.**
   - Generate another Spring Boot project called `OrderService`:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
   - Add `OrderController.java` in `src/main/java/com/microservices/orderservice`:
     ```java
     package com.microservices.orderservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class OrderController {

         @GetMapping("/orders")
         public String getOrders() {
             return "List of orders";
         }
     }
     ```

7. **Run the `OrderService` application.**
   - Start `OrderService` on port `8081`. Add `application.properties`:
     ```properties
     server.port=8081
     ```

8. **Configure `UserService` to call `OrderService`.**
   - Add a `RestTemplate` bean in `UserServiceApplication.java`:
     ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.context.annotation.Bean;
     import org.springframework.web.client.RestTemplate;

     @SpringBootApplication
     public class UserServiceApplication {
         public static void main(String[] args) {
             SpringApplication.run(UserServiceApplication.class, args);
         }

         @Bean
         public RestTemplate restTemplate() {
             return new RestTemplate();
         }
     }
     ```

9. **Modify `UserController` to include a circuit breaker.**
   - Update `UserController.java`:
     ```java
     package com.microservices.userservice;

     import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;
     import org.springframework.web.client.RestTemplate;

     @RestController
     public class UserController {

         @Autowired
         private RestTemplate restTemplate;

         @CircuitBreaker(name = "orderService", fallbackMethod = "fallbackOrders")
         @GetMapping("/user-orders")
         public String getUserOrders() {
             String orders = restTemplate.getForObject("http://localhost:8081/orders", String.class);
             return "User Orders: " + orders;
         }

         public String fallbackOrders(Throwable throwable) {
             return "Fallback: No orders available";
         }
     }
     ```

10. **Add Resilience4j configurations.**
    - Create `application.properties`:
      ```properties
      resilience4j.circuitbreaker.instances.orderService.slidingWindowSize=10
      resilience4j.circuitbreaker.instances.orderService.failureRateThreshold=50
      resilience4j.circuitbreaker.instances.orderService.waitDurationInOpenState=10s
      ```

11. **Test the `/user-orders` endpoint.**
    - Start both `UserService` and `OrderService`.
    - Access:
      ```
      http://localhost:8080/user-orders
      ```
    - Verify the response: `"User Orders: List of orders"`.

12. **Simulate a failure in `OrderService`.**
    - Stop `OrderService`.
    - Re-access `/user-orders`.
    - Verify the fallback response: `"Fallback: No orders available"`.

---

### **Part 3: Monitoring Circuit Breakers**

13. **Add Spring Boot Actuator configurations.**
    - Update `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=resilience4j.circuitbreakers
      ```

14. **Monitor circuit breaker metrics.**
    - Access:
      ```
      http://localhost:8080/actuator/resilience4j/circuitbreakers
      ```
    - Verify the current state of the `orderService` circuit breaker.

---

### **Optional Exercises**

1. **Add more fallback scenarios.**
   - Add another endpoint in `OrderService` (e.g., `/order-details`) and configure a fallback in `UserService`.

2. **Test timeout configurations.**
   - Add custom timeout settings in `application.properties`:
     ```properties
     resilience4j.timeout.instances.orderService.timeoutDuration=2s
     ```
   - Simulate long response times in `OrderService` and verify the fallback behavior.

3. **Visualize circuit breaker health with external tools.**
   - Integrate Resilience4j with Prometheus and Grafana for detailed monitoring.

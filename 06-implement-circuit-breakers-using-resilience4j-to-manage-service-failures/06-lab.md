# **Lab 6: Implement Circuit Breakers Using Resilience4j to Manage Service Failures**

## **Objective**
Learn how to implement circuit breakers for fault tolerance using Resilience4j (the recommended replacement for Hystrix). Configure fallback methods to handle service failures gracefully and monitor service health.

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
   - Extract the downloaded zip file into a folder named `UserService`.

2. **Import the project into your IDE.**
   - Open your IDE (e.g., IntelliJ, Eclipse, or VS Code) and import the `UserService` project as a Maven project.

3. **Add a REST controller to simulate a user endpoint.**
   - Create a new file `UserController.java` in the `src/main/java/com/microservices/userservice` folder:
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
   - Open a browser or use Postman to access:
     ```
     http://localhost:8080/users
     ```
   - Verify that the response is `"List of users"`.

---

### **Part 2: Adding a Circuit Breaker**

6. **Simulate a dependent service.**
   - Generate another Spring Boot project called `OrderService` with the following configuration:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
   - Add a REST endpoint in `OrderService` to simulate a dependency:
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
   - Start `OrderService` on port `8081` by adding the following in `src/main/resources/application.yml`:
     ```yaml
     server:
       port: 8081
     ```

8. **Configure the `UserService` to call `OrderService`.**
   - Add a `RestTemplate` bean in the `UserServiceApplication.java` file:
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

9. **Modify the `UserController` to call `OrderService`.**
   - Update `UserController` to include a `/user-orders` endpoint with a circuit breaker:
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
    - Create `application.yml` in `src/main/resources`:
      ```yaml
      resilience4j:
        circuitbreaker:
          configs:
            default:
              registerHealthIndicator: true
              slidingWindowSize: 10
              failureRateThreshold: 50
              waitDurationInOpenState: 10s
          instances:
            orderService:
              base-config: default
      ```

11. **Test the `/user-orders` endpoint.**
    - Start both `UserService` and `OrderService`.
    - Access:
      ```
      http://localhost:8080/user-orders
      ```
    - Verify that it returns `"User Orders: List of orders"`.

12. **Simulate a failure in `OrderService`.**
    - Stop the `OrderService` application.
    - Re-access the `/user-orders` endpoint.
    - Verify that the fallback response `"Fallback: No orders available"` is returned.

---

### **Part 3: Monitoring Circuit Breakers**

13. **Add Spring Boot Actuator configurations.**
    - Add the following to `application.yml`:
      ```yaml
      management:
        endpoints:
          web:
            exposure:
              include: resilience4j.circuitbreakers
      ```

14. **Monitor circuit breaker status.**
    - Access the circuit breaker metrics:
      ```
      http://localhost:8080/actuator/resilience4j/circuitbreakers
      ```
    - Verify the current state of the `orderService` circuit breaker.

---

### **Optional Exercises**

1. **Add more fallback scenarios.**
   - Introduce additional dependent endpoints in `OrderService` and create corresponding fallbacks in `UserService`.

2. **Configure timeout thresholds.**
   - Add custom timeout configurations in `application.yml` and test:
     ```yaml
     resilience4j:
       timeout:
         default:
           timeoutDuration: 2s
     ```

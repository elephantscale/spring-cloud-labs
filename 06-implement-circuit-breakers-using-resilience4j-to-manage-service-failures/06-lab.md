# **Lab 6: Implement Circuit Breakers Using Resilience4j to Manage Service Failures (Spring Boot 3.4.1)**

## **Objective**
Learn how to implement circuit breakers for fault tolerance using **Resilience4j** with **Spring Boot 3.4.1**. Configure fallback methods to handle service failures gracefully and monitor service health in real time.

---

## **Lab Steps**

### **Part 1: Setting Up the Base Service**

1. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Resilience4j Spring Boot Starter
   - Click **Generate** to download the project zip file.
   - Extract the zip into a folder named `UserService`.

2. **Import the project into your IDE.**
   - Open your IDE and import `UserService` as a Maven project.

3. **Add a REST controller to simulate a user endpoint.**
   - In `src/main/java/com/microservices/userservice/UserController.java`:
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
   - From the `UserService` directory:
     ```bash
     ./mvnw spring-boot:run
     ```

5. **Test the `/users` endpoint.**
   - In a browser or via curl, access:
     ```
     http://localhost:8080/users
     ```
   - Confirm the response is `"List of users"`.

---

### **Part 2: Adding a Circuit Breaker**

6. **Simulate a dependent service.**
   - Create another project, `OrderService`, the same way. Choose:
     - **Artifact Id**: `order-service`
     - **Dependencies**: 
       - Spring Web
   - In `src/main/java/com/microservices/orderservice/OrderController.java`:
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
   - In `OrderService`’s `application.properties`:
     ```properties
     server.port=8081
     ```

7. **Configure `UserService` to call `OrderService`.**
   - In `UserServiceApplication.java`, add a `RestTemplate` bean:
     ```java
     package com.microservices.userservice;

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

8. **Add a circuit breaker to the controller.**
   - Modify `UserController.java`:
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

9. **Add Resilience4j configurations.**
   - In `application.properties`:
     ```properties
     resilience4j.circuitbreaker.instances.orderService.slidingWindowSize=10
     resilience4j.circuitbreaker.instances.orderService.failureRateThreshold=50
     resilience4j.circuitbreaker.instances.orderService.waitDurationInOpenState=10s
     ```

10. **Test the `/user-orders` endpoint.**
    - Start both `OrderService` (on port **8081**) and `UserService` (on port **8080**).
    - Access:
      ```
      http://localhost:8080/user-orders
      ```
    - Should respond: `"User Orders: List of orders"`.

11. **Simulate a failure in `OrderService`.**
    - Stop `OrderService`.
    - Re-access `/user-orders`.
    - Verify fallback: `"Fallback: No orders available"`.

---

### **Part 3: Monitoring Circuit Breakers**

12. **Add Spring Boot Actuator configs.**
    - In `application.properties` (UserService):
      ```properties
      management.endpoints.web.exposure.include=resilience4j.circuitbreakers
      ```

13. **Check circuit breaker metrics.**
    - Access:
      ```
      http://localhost:8080/actuator/resilience4j/circuitbreakers
      ```
    - Observe the circuit breaker state, failure rates, etc.

---

### **Optional Exercises**

1. **Add more fallback scenarios.**
   - Create `/order-details` in `OrderService` and add a second circuit breaker call in `UserService`.

2. **Configure timeout thresholds.**
   - E.g., in `application.properties`:
     ```properties
     resilience4j.timeout.instances.orderService.timeoutDuration=2s
     ```
   - Simulate delays in `OrderService` to trigger timeouts.

3. **Visualize with external tools.**
   - Integrate **Resilience4j** with Prometheus and Grafana for robust dashboards and alerts.

---

## **Conclusion**
By completing this lab, you have successfully:
- **Implemented circuit breakers** using **Resilience4j** and Spring Boot **3.4.1**.
- **Protected** a `UserService` from `OrderService` failures using a fallback method.
- **Monitored** circuit breaker states via Spring Boot Actuator endpoints.
- **Simulated failures** to confirm fallback logic and validated your microservice’s resiliency to dependency outages. Enjoy building more robust microservices!

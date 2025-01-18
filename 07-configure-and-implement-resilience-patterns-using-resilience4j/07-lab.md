# **Lab 7: Configure and Implement Resilience Patterns Using Resilience4j**

## **Objective**
Learn how to implement resilience patterns, including circuit breakers, retries, rate limiting, and bulkhead isolation, using Resilience4j. Explore how these patterns improve fault tolerance and system stability.

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

3. **Add Resilience4j configurations to `application.properties`.**
   - Create `application.properties` in `src/main/resources` and add:
     ```properties
     resilience4j.circuitbreaker.instances.user-service.failure-rate-threshold=50
     resilience4j.circuitbreaker.instances.user-service.sliding-window-size=5
     resilience4j.circuitbreaker.instances.user-service.wait-duration-in-open-state=5s
     ```

4. **Add a REST controller to simulate a user endpoint.**
   - Create `UserController.java` in `src/main/java/com/microservices/userservice`:
     ```java
     package com.microservices.userservice;

     import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class UserController {

         @GetMapping("/users")
         @CircuitBreaker(name = "user-service", fallbackMethod = "fallbackGetUsers")
         public String getUsers() {
             simulateServiceFailure();
             return "List of users";
         }

         public String fallbackGetUsers(Throwable t) {
             return "Fallback: No users available";
         }

         private void simulateServiceFailure() {
             throw new RuntimeException("Simulated service failure");
         }
     }
     ```

5. **Run the application.**
   - Start the application:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test the `/users` endpoint.**
   - Use Postman or a browser to access:
     ```
     http://localhost:8080/users
     ```
   - Verify the fallback response: `"Fallback: No users available"`.

---

### **Part 2: Adding Retry Mechanism**

7. **Configure retries in `application.properties`.**
   - Add:
     ```properties
     resilience4j.retry.instances.user-service.max-attempts=3
     resilience4j.retry.instances.user-service.wait-duration=500ms
     ```

8. **Add retry to the `UserController`.**
   - Update the `@GetMapping` method:
     ```java
     import io.github.resilience4j.retry.annotation.Retry;

     @GetMapping("/users")
     @CircuitBreaker(name = "user-service", fallbackMethod = "fallbackGetUsers")
     @Retry(name = "user-service")
     public String getUsers() {
         simulateServiceFailure();
         return "List of users";
     }
     ```

9. **Test the retry mechanism.**
   - Access the `/users` endpoint.
   - Verify that the service retries up to 3 times before falling back.

---

### **Part 3: Adding Rate Limiting**

10. **Configure rate limiting in `application.properties`.**
    - Add:
      ```properties
      resilience4j.ratelimiter.instances.user-service.limit-for-period=2
      resilience4j.ratelimiter.instances.user-service.limit-refresh-period=10s
      ```

11. **Add rate limiting to the `UserController`.**
    - Update the `@GetMapping` method:
      ```java
      import io.github.resilience4j.ratelimiter.annotation.RateLimiter;

      @GetMapping("/users")
      @CircuitBreaker(name = "user-service", fallbackMethod = "fallbackGetUsers")
      @Retry(name = "user-service")
      @RateLimiter(name = "user-service")
      public String getUsers() {
          simulateServiceFailure();
          return "List of users";
      }
      ```

12. **Test rate limiting.**
    - Call the `/users` endpoint multiple times within 10 seconds.
    - Verify that after 2 requests, subsequent calls are blocked.

---

### **Part 4: Adding Bulkhead Isolation**

13. **Configure bulkhead isolation in `application.properties`.**
    - Add:
      ```properties
      resilience4j.bulkhead.instances.user-service.max-concurrent-calls=2
      resilience4j.bulkhead.instances.user-service.max-wait-duration=0
      ```

14. **Add bulkhead isolation to the `UserController`.**
    - Update the `@GetMapping` method:
      ```java
      import io.github.resilience4j.bulkhead.annotation.Bulkhead;

      @GetMapping("/users")
      @CircuitBreaker(name = "user-service", fallbackMethod = "fallbackGetUsers")
      @Retry(name = "user-service")
      @RateLimiter(name = "user-service")
      @Bulkhead(name = "user-service")
      public String getUsers() {
          simulateServiceFailure();
          return "List of users";
      }
      ```

15. **Test bulkhead isolation.**
    - Simulate multiple concurrent calls to the `/users` endpoint.
    - Verify that only 2 concurrent requests are allowed.

---

### **Part 5: Monitoring with Resilience4j**

16. **Expose Actuator endpoints.**
    - Add to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=health,metrics
      ```

17. **Access Resilience4j metrics.**
    - Start the application and access:
      ```
      http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.calls
      ```

18. **Test monitoring under load.**
    - Simulate requests to `/users` and observe metrics for retries, rate limiting, and bulkhead.

---

### **Optional Exercises**

1. **Test Circuit Breaker Recovery.**
   - Allow the circuit breaker to reset after `wait-duration-in-open-state`.
   - Verify the service resumes normal operation.

2. **Simulate Rate Limit Breaches.**
   - Call `/users` frequently to observe rate limiter behavior.

3. **Test Bulkhead Under Stress.**
   - Simulate high concurrency to verify isolation limits.

---

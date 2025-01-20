# **Lab 7: Configure and Implement Resilience Patterns Using Resilience4j (Spring Boot 3.4.1)**

## **Objective**
Learn how to integrate **Resilience4j** into a Spring Boot **3.4.1** application to implement multiple resilience patterns:
- **Circuit Breaker**: Handle and recover from dependency failures
- **Retry**: Automatically retry failed operations
- **Rate Limiter**: Control request throughput
- **Bulkhead**: Isolate resources to avoid cascading failures

You will configure these patterns and verify how they improve fault tolerance in your microservices.

---

## **Lab Steps**

### **Part 1: Setting Up the Base Service**

1. **Generate a new Spring Boot project for `UserService`.**
   - Go to [https://start.spring.io/](https://start.spring.io/).
   - **Spring Boot Version**: **3.4.1**
   - **Group Id**: `com.microservices`
   - **Artifact Id**: `user-service`
   - **Name**: `UserService`
   - **Dependencies**:
     - Spring Web
     - Spring Boot Actuator
     - Resilience4j Spring Boot Starter
   - Click **Generate** and extract the project into a folder named `UserService`.

2. **Import the project into your IDE.**
   - Import `UserService` as a Maven project.

3. **Configure Resilience4j in `application.properties`.**
   - Create or update `src/main/resources/application.properties`:
     ```properties
     # Initial circuit breaker settings for "user-service"
     resilience4j.circuitbreaker.instances.user-service.failure-rate-threshold=50
     resilience4j.circuitbreaker.instances.user-service.sliding-window-size=5
     resilience4j.circuitbreaker.instances.user-service.wait-duration-in-open-state=5s
     ```
   - These values will be used by the **Circuit Breaker**.

4. **Add a REST controller to simulate a user endpoint.**
   - In `src/main/java/com/microservices/userservice/UserController.java`:
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
   - This initial code sets up a **circuit breaker** on `getUsers()` that falls back to `fallbackGetUsers()` if something goes wrong.

5. **Run the application.**
   - From the `UserService` folder:
     ```bash
     ./mvnw spring-boot:run
     ```
   - The service starts on `http://localhost:8080` by default.

6. **Test the `/users` endpoint.**
   - In a browser or via Postman/cURL:
     ```
     http://localhost:8080/users
     ```
   - The call triggers `simulateServiceFailure()`, raising an exception. The circuit breaker uses `fallbackGetUsers()`, returning `"Fallback: No users available"`.

---

### **Part 2: Adding a Retry Mechanism**

7. **Configure retry in `application.properties`.**
   - Add:
     ```properties
     # Retry settings for "user-service"
     resilience4j.retry.instances.user-service.max-attempts=3
     resilience4j.retry.instances.user-service.wait-duration=500ms
     ```
   - Now, the method will try to invoke the operation multiple times before the circuit breaker logic sees it as a failure.

8. **Add the `@Retry` annotation to your method.**
   - Update `UserController`:
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
   - This means each request to `/users` will be retried up to 3 times with a 500ms wait between attempts.

9. **Verify the retry mechanism.**
   - Access `/users` again. Observe in logs or debugging that the **simulateServiceFailure()** is called multiple times before the fallback is used.

---

### **Part 3: Adding Rate Limiting**

10. **Configure rate limiting in `application.properties`.**
    - Add:
      ```properties
      # Rate limiter for "user-service"
      resilience4j.ratelimiter.instances.user-service.limit-for-period=2
      resilience4j.ratelimiter.instances.user-service.limit-refresh-period=10s
      ```
    - This configuration allows only 2 requests in each 10-second window.

11. **Add `@RateLimiter` to the method.**
    - Modify `UserController`:
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
    - Once the limit (2 requests) is hit, further requests are blocked until the refresh period (10s) elapses.

12. **Test rate limiting.**
    - Hit the `/users` endpoint more than 2 times within 10 seconds.
    - The 3rd+ requests should be blocked or produce a `RateLimitExceededException`, leading to a fallback or error.

---

### **Part 4: Adding Bulkhead Isolation**

13. **Configure bulkhead isolation in `application.properties`.**
    - Add:
      ```properties
      # Bulkhead settings for "user-service"
      resilience4j.bulkhead.instances.user-service.max-concurrent-calls=2
      resilience4j.bulkhead.instances.user-service.max-wait-duration=0
      ```
    - This allows only 2 concurrent calls at a time.

14. **Add `@Bulkhead` to the method.**
    - In `UserController`:
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
    - If more than 2 concurrent requests come in, additional calls immediately fail or get fallback.

15. **Test bulkhead isolation.**
    - Send multiple concurrent `/users` requests (e.g. with a load testing tool or multiple browser tabs).
    - Confirm that only 2 requests are processed concurrently.

---

### **Part 5: Monitoring with Resilience4j**

16. **Expose Actuator endpoints.**
    - In `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=health,metrics
      ```
    - This ensures you can view metrics related to circuit breakers, retries, rate limiters, and bulkheads.

17. **Access Resilience4j metrics.**
    - Start or restart the service, then go to:
      ```
      http://localhost:8080/actuator/metrics
      ```
    - Look for `resilience4j.circuitbreaker.calls`, `resilience4j.retry.calls`, etc.

18. **Test under load.**
    - Send multiple requests to `/users` to trigger rate limits, retries, circuit breaker flips, etc.
    - Check logs and actuator endpoints to confirm the resilience patterns are working.

---

## **Optional Exercises**

1. **Test Circuit Breaker Recovery.**
   - Once the circuit is open, wait for `wait-duration-in-open-state` to elapse. Observe it transitioning to half-open, then closed if successful calls are made.

2. **Simulate Rate Limit Breaches.**
   - Fire more requests than the configured `limit-for-period` within one refresh window and observe fallback or blocking behavior.

3. **Simulate Bulkhead Under Stress.**
   - Use concurrent threads or a load testing tool to see how extra calls over `max-concurrent-calls` get rejected or fallback.

4. **Integrate with Monitoring Tools.**
   - Combine Prometheus/Grafana or other dashboards to visualize resilience metrics in real-time.

---

## **Conclusion**
By completing this lab, you:
- **Implemented multiple Resilience4j patterns**: Circuit Breaker, Retry, Rate Limiter, and Bulkhead.
- **Configured** them all on a single method (`/users`) in your **Spring Boot 3.4.1** service.
- **Observed** how each pattern mitigates different failure/overload scenarios.

You now have a robust template for building highly resilient microservices capable of handling partial failures, traffic spikes, and concurrency limits gracefully!

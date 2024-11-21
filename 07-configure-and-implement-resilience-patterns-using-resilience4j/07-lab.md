# **Lab 7: Configure and Implement Resilience Patterns Using Resilience4j**

## **Objective**
Learn how to implement resilience patterns, including circuit breakers, retries, rate limiting, and bulkhead isolation, using Resilience4j. Explore how these patterns improve fault tolerance and system stability.

---

## **Lab Steps**

### **Part 1: Setting Up the Base Service**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
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
   - Open your IDE and import the `UserService` project as a Maven project.

3. **Add Resilience4j configurations to `application.properties`.**
   - Open the `src/main/resources/application.properties` file and add:
     ```properties
     resilience4j.circuitbreaker.instances.user-service.failure-rate-threshold=50
     resilience4j.circuitbreaker.instances.user-service.sliding-window-size=5
     resilience4j.circuitbreaker.instances.user-service.wait-duration-in-open-state=5s
     ```

4. **Add a REST controller to simulate a user endpoint.**
   - Create a new file `UserController.java` in the `src/main/java/com/microservices/userservice` folder:
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
   - Start the `UserService` application using:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test the `/users` endpoint.**
   - Use Postman or a browser to access:
     ```
     http://localhost:8080/users
     ```
   - Verify that the fallback response `"Fallback: No users available"` is returned after triggering the circuit breaker.

---

### **Part 2: Adding Retry Mechanism**

7. **Configure retries in `application.properties`.**
   - Add the following Resilience4j retry properties:
     ```properties
     resilience4j.retry.instances.user-service.max-attempts=3
     resilience4j.retry.instances.user-service.wait-duration=500ms
     ```

8. **Update the `UserController` to include a retry mechanism.**
   - Add the `@Retry` annotation to the `getUsers` method:
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
   - Use Postman or a browser to call the `/users` endpoint.
   - Verify that the service attempts to retry the method up to three times before falling back to the fallback method.

---

### **Part 3: Adding Rate Limiting**

10. **Configure rate limiting in `application.properties`.**
    - Add the following Resilience4j rate limiter properties:
      ```properties
      resilience4j.ratelimiter.instances.user-service.limit-for-period=2
      resilience4j.ratelimiter.instances.user-service.limit-refresh-period=10s
      ```

11. **Add the rate limiter to the `UserController`.**
    - Update the `getUsers` method to include the `@RateLimiter` annotation:
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
    - Call the `/users` endpoint multiple times within a 10-second window.
    - Verify that after 2 requests, subsequent calls are blocked until the refresh period ends.

---

### **Part 4: Adding Bulkhead Isolation**

13. **Configure bulkhead isolation in `application.properties`.**
    - Add the following Resilience4j bulkhead properties:
      ```properties
      resilience4j.bulkhead.instances.user-service.max-concurrent-calls=2
      resilience4j.bulkhead.instances.user-service.max-wait-duration=0
      ```

14. **Add the bulkhead to the `UserController`.**
    - Update the `getUsers` method to include the `@Bulkhead` annotation:
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
    - Verify that only 2 concurrent requests are allowed, and additional requests are immediately rejected.

---

### **Part 5: Monitoring with Resilience4j**

16. **Add Spring Boot Actuator dependency.**
    - Ensure the following dependency is added in `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose Actuator endpoints.**
    - Add the following to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=health,metrics
      ```

18. **Access Resilience4j metrics.**
    - Start the application and access:
      ```
      http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.calls
      ```
    - Verify that metrics for circuit breaker calls are displayed.

19. **Test monitoring under load.**
    - Simulate multiple requests to the `/users` endpoint and observe the metrics for retries, rate limiting, and bulkhead.

---

### **Optional Exercises (20 mins)**

1. **Test circuit breaker recovery.**
   - Allow the circuit breaker to reset by waiting for the `wait-duration-in-open-state` to elapse.
   - Test if the service resumes normal operation.

2. **Create a custom fallback method.**
   - Implement different fallback methods for various failure scenarios and test their behavior.

---

### **Dependencies**
The verified dependencies for this lab are:

```xml
<dependencies>
    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Resilience4j -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot3</artifactId>
    </dependency>

    <!-- Spring Boot Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

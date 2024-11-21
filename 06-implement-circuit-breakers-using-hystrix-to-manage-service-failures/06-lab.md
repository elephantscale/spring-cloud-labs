# **Lab 6: Implement Circuit Breakers Using Hystrix to Manage Service Failures**

## **Objective**
Learn how to use Hystrix to implement circuit breakers for fault tolerance. Configure fallback methods to handle service failures gracefully and monitor service health using the Hystrix Dashboard.

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
       - Spring Cloud Netflix Hystrix
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `UserService`.

2. **Import the project into your IDE.**
   - Open your favorite IDE and import the `UserService` project as a Maven project.

3. **Enable Hystrix in the application.**
   - Open the `UserServiceApplication.java` file in the `src/main/java/com/microservices/userservice` folder.
   - Add the `@EnableCircuitBreaker` annotation:
     ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;

     @SpringBootApplication
     @EnableCircuitBreaker
     public class UserServiceApplication {
         public static void main(String[] args) {
             SpringApplication.run(UserServiceApplication.class, args);
         }
     }
     ```

4. **Add a REST controller to simulate a user endpoint.**
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

5. **Run the application.**
   - Start the `UserService` application using:
     ```bash
     ./mvnw spring-boot:run
     ```

6. **Test the `/users` endpoint.**
   - Open a browser or use Postman to access:
     ```
     http://localhost:8080/users
     ```
   - Verify that the response returns `"List of users"`.

---

### **Part 2: Adding a Circuit Breaker**

7. **Simulate a dependent service.**
   - Generate another Spring Boot project called `OrderService` with the same steps as above, but configure:
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

8. **Run the `OrderService` application.**
   - Start `OrderService` on port `8081` by adding the following to `application.properties`:
     ```properties
     server.port=8081
     ```

9. **Configure the `UserService` to call `OrderService`.**
   - Add the following `RestTemplate` bean in the `UserServiceApplication.java`:
     ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
     import org.springframework.context.annotation.Bean;
     import org.springframework.web.client.RestTemplate;

     @SpringBootApplication
     @EnableCircuitBreaker
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

10. **Modify the `UserController` to call `OrderService`.**
    - Update `UserController` to include a `/user-orders` endpoint:
      ```java
      package com.microservices.userservice;

      import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;
      import org.springframework.web.client.RestTemplate;

      @RestController
      public class UserController {

          @Autowired
          private RestTemplate restTemplate;

          @HystrixCommand(fallbackMethod = "fallbackOrders")
          @GetMapping("/user-orders")
          public String getUserOrders() {
              String orders = restTemplate.getForObject("http://localhost:8081/orders", String.class);
              return "User Orders: " + orders;
          }

          public String fallbackOrders() {
              return "Fallback: No orders available";
          }
      }
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

### **Part 3: Monitoring with Hystrix Dashboard**

13. **Add Hystrix Dashboard dependency.**
    - Add the following to the `UserService` `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
      </dependency>
      ```

14. **Enable the Hystrix Dashboard.**
    - Add the `@EnableHystrixDashboard` annotation in the `UserServiceApplication`:
      ```java
      import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

      @SpringBootApplication
      @EnableCircuitBreaker
      @EnableHystrixDashboard
      public class UserServiceApplication {
          // Other code remains the same
      }
      ```

15. **Configure Hystrix Dashboard properties.**
    - Add the following in `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=hystrix.stream
      ```

16. **Run the Hystrix Dashboard.**
    - Restart the `UserService` application.
    - Open the dashboard:
      ```
      http://localhost:8080/hystrix
      ```

17. **Monitor circuit breaker activity.**
    - Enter the stream URL:
      ```
      http://localhost:8080/actuator/hystrix.stream
      ```
    - Click **Monitor Stream**.

18. **Test the dashboard.**
    - Trigger requests to `/user-orders` with and without `OrderService` running.
    - Observe how the circuit breaker status changes on the dashboard.

---

### **Optional Exercises (20 mins)**

1. **Add more fallback scenarios.**
   - Introduce additional dependent endpoints in `OrderService` and create corresponding fallbacks in `UserService`.

2. **Configure timeout thresholds.**
   - Add Hystrix timeout configurations in `application.properties` and test:
     ```properties
     hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=2000
     ```

---

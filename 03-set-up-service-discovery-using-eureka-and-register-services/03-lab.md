# **Lab 3: Set Up Service Discovery Using Eureka and Register Services (Spring Boot 3.4.1)**

## **Objective**
Learn how to set up a **Eureka Server** for service discovery and enable multiple microservices (Spring Boot **3.4.1**) to register dynamically. Test discovery and interaction between services using Eureka.

---

## **Lab Steps**

### **Part 1: Setting Up the Eureka Server**

1. **Generate a new Spring Boot project for `EurekaServer`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `eureka-server`
     - **Name**: `EurekaServer`
     - **Package Name**: `com.microservices.eurekaserver`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Eureka Server
   - Click **Generate** to download the project zip file.
   - Extract it into a folder named `EurekaServer`.

2. **Import the project into your IDE.**
   - Open your IDE (e.g., IntelliJ, Eclipse, VS Code) and import `EurekaServer` as a Maven project.

3. **Enable the Eureka Server.**
   - In `EurekaServerApplication.java` (inside `src/main/java/com/microservices/eurekaserver`):
     ```java
     package com.microservices.eurekaserver;

     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

     @SpringBootApplication
     @EnableEurekaServer
     public class EurekaServerApplication {
         public static void main(String[] args) {
             SpringApplication.run(EurekaServerApplication.class, args);
         }
     }
     ```

4. **Configure the Eureka Server in `application.properties`.**
   - In `src/main/resources/application.properties`, add:
     ```properties
     spring.application.name=eureka-server
     server.port=8761

     eureka.client.register-with-eureka=false
     eureka.client.fetch-registry=false
     ```

5. **Run the Eureka Server.**
   - From the `EurekaServer` directory:
     ```bash
     ./mvnw spring-boot:run
     ```
   - Check the Eureka Dashboard at:
     ```
     http://localhost:8761
     ```
   - You should see **Eureka** up and running, but with no registered services yet.

---

### **Part 2: Creating a Client Service (UserService)**

6. **Generate a new Spring Boot project for `UserService`.**
   - Repeat the process at [https://start.spring.io/](https://start.spring.io/):
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Package Name**: `com.microservices.userservice`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Eureka Client
   - Extract the zip into `UserService`.

7. **Import `UserService` into your IDE.**

8. **Enable Eureka Client in `UserService`.**
   - In `UserServiceApplication.java`:
     ```java
     package com.microservices.userservice;

     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

     @SpringBootApplication
     @EnableEurekaClient
     public class UserServiceApplication {
         public static void main(String[] args) {
             SpringApplication.run(UserServiceApplication.class, args);
         }
     }
     ```

9. **Configure Eureka in `application.properties`.**
   - In `src/main/resources/application.properties`, add:
     ```properties
     spring.application.name=user-service
     server.port=8081

     eureka.client.service-url.defaultZone=http://localhost:8761/eureka
     ```

10. **Add a REST endpoint.**
    - Create `UserController.java` in `com.microservices.userservice`:
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

11. **Run the `UserService`.**
    - From the `UserService` directory:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check the logs. It should say `Registering application user-service with Eureka...`

12. **Verify service registration.**
    - Refresh the Eureka Dashboard at:
      ```
      http://localhost:8761
      ```
    - You should see `user-service` registered.

---

### **Part 3: Adding Another Service (ProductService)**

13. **Generate a new Spring Boot project for `ProductService`.**
   - Same approach:
     - **Artifact Id**: `product-service`
     - **Name**: `ProductService`
     - **Package Name**: `com.microservices.productservice`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Eureka Client

14. **Enable Eureka Client.**
   - In `ProductServiceApplication.java`:
     ```java
     package com.microservices.productservice;

     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

     @SpringBootApplication
     @EnableEurekaClient
     public class ProductServiceApplication {
         public static void main(String[] args) {
             SpringApplication.run(ProductServiceApplication.class, args);
         }
     }
     ```

15. **Configure Eureka in `application.properties`.**
   - In `src/main/resources/application.properties`:
     ```properties
     spring.application.name=product-service
     server.port=8082

     eureka.client.service-url.defaultZone=http://localhost:8761/eureka
     ```

16. **Add a REST endpoint for `ProductService`.**
   - Create `ProductController.java`:
     ```java
     package com.microservices.productservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class ProductController {

         @GetMapping("/products")
         public String getProducts() {
             return "List of products";
         }
      }
      ```

17. **Run the `ProductService`.**
    - Again:
      ```bash
      ./mvnw spring-boot:run
      ```

18. **Verify both services in Eureka.**
    - Check `http://localhost:8761`.
    - You should see both `user-service` and `product-service` registered.

---

### **Part 4: Testing Service Discovery**

19. **Test microservices endpoints.**
    - `UserService`: 
      ```
      http://localhost:8081/users
      ```
    - `ProductService`:
      ```
      http://localhost:8082/products
      ```
    - Both should return simple strings verifying they’re live and registered with Eureka.

20. **(Optional) Test inter-service calls.**
    - You can call `http://user-service/users` from `ProductService` if needed, using Eureka’s naming, but that involves Ribbon or Spring LoadBalancer for dynamic discovery.

---

## **Conclusion**
By completing this lab, you have:
- **Set up a Eureka Server** (Spring Boot **3.4.1**) on port 8761.
- **Registered two microservices** (`UserService`, `ProductService`) with Eureka.
- **Verified** that Eureka’s dashboard shows these services as **UP**.
- **Optionally tested** basic REST endpoints.

This enables dynamic discovery of services without hardcoded URLs, forming the basis for microservice interaction.

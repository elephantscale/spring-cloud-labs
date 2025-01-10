# **Lab 3: Set Up Service Discovery Using Eureka and Register Services**

## **Objective**
Learn how to set up a Eureka Server for service discovery and enable multiple microservices to register dynamically. Test service discovery and interaction between services using Eureka.

---

## **Lab Steps**

### **Part 1: Setting Up the Eureka Server**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `eureka-server`
     - **Name**: `EurekaServer`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Eureka Server
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `EurekaServer`.

2. **Import the project into your IDE.**
   - Open your IDE (e.g., IntelliJ, Eclipse, or VS Code).
   - Import the `EurekaServer` project as a Maven project.
   - Ensure all dependencies are downloaded successfully.

3. **Enable the Eureka Server.**
   - Open the `EurekaServerApplication.java` file in the `src/main/java/com/microservices/eurekaserver` folder.
   - Add the `@EnableEurekaServer` annotation:
     ```java
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

4. **Configure the Eureka Server in `application.yml`.**
   - Create a file named `application.yml` in the `src/main/resources` folder and add the following configuration:
     ```yaml
     spring:
       application:
         name: eureka-server

     server:
       port: 8761

     eureka:
       client:
         register-with-eureka: false
         fetch-registry: false
     ```

5. **Run the Eureka Server.**
   - Start the `EurekaServer` application:
     - **Windows**:
       ```cmd
       mvnw.cmd spring-boot:run
       ```
     - **macOS/Linux**:
       ```bash
       ./mvnw spring-boot:run
       ```

6. **Access the Eureka Dashboard.**
   - Open a browser and navigate to:
     ```
     http://localhost:8761
     ```
   - Verify that the Eureka dashboard is displayed with no registered services yet.

---

### **Part 2: Setting Up a Client Service**

7. **Generate a new Spring Boot project using Spring Initializr.**
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Eureka Client
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `UserService`.

8. **Import the project into your IDE.**

9. **Enable Eureka Client in the application.**
   - Open the `UserServiceApplication.java` file in the `src/main/java/com/microservices/userservice` folder.
   - Add the `@EnableEurekaClient` annotation:
     ```java
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

10. **Configure the Eureka client in `application.yml`.**
    - Create a file named `application.yml` in the `src/main/resources` folder and add the following:
      ```yaml
      spring:
        application:
          name: user-service

      server:
        port: 8081

      eureka:
        client:
          service-url:
            defaultZone: http://localhost:8761/eureka
      ```

11. **Add a sample REST endpoint to the service.**
    - Create `UserController.java` in the `src/main/java/com/microservices/userservice` folder:
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

12. **Run the `UserService` application.**
    - Start the service:
      - **Windows**:
        ```cmd
        mvnw.cmd spring-boot:run
        ```
      - **macOS/Linux**:
        ```bash
        ./mvnw spring-boot:run
        ```

13. **Verify service registration in Eureka.**
    - Refresh the Eureka dashboard at `http://localhost:8761`.
    - Confirm that `user-service` is listed as a registered service.

---

### **Part 3: Adding Another Service**

14. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 7 and 8 with the following changes:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`

15. **Enable Eureka Client for `ProductService`.**
    - Add the `@EnableEurekaClient` annotation in the `ProductServiceApplication.java` file.

16. **Configure Eureka for `ProductService`.**
    - Create a file named `application.yml` in the `src/main/resources` folder and add:
      ```yaml
      spring:
        application:
          name: product-service

      server:
        port: 8082

      eureka:
        client:
          service-url:
            defaultZone: http://localhost:8761/eureka
      ```

17. **Add a REST endpoint for the `ProductService`.**
    - Create `ProductController.java` in the `src/main/java/com/microservices/productservice` folder:
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

18. **Run the `ProductService` application.**
    - Start the service:
      - **Windows**:
        ```cmd
        mvnw.cmd spring-boot:run
        ```
      - **macOS/Linux**:
        ```bash
        ./mvnw spring-boot:run
        ```

19. **Verify service registration in Eureka.**
    - Refresh the Eureka dashboard at `http://localhost:8761`.
    - Confirm that both `user-service` and `product-service` are listed.

---

### **Part 4: Testing Service Discovery**

20. **Test inter-service communication using Eureka.**
    - Test the REST endpoints:
      - `UserService`: `http://localhost:8081/users`
      - `ProductService`: `http://localhost:8082/products`

---

### **Optional Exercises**

1. **Run multiple instances of `UserService`.**
   - Modify `application.yml` to run on different ports (e.g., `8081`, `8083`).
   - Verify multiple instances in Eureka.

2. **Integrate Ribbon for client-side load balancing.**
   - Add Ribbon dependency to a client microservice and test load balancing between `UserService` instances.

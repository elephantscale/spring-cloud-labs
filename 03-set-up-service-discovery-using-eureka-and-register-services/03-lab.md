# **Lab 3: Set Up Service Discovery Using Eureka and Register Services**

## **Objective**
Learn how to set up a Eureka Server for service discovery and enable multiple microservices to register dynamically. Test service discovery and interaction between services using Eureka.

---

## **Lab Steps**

### **Part 1: Setting Up the Eureka Server**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `eureka-server`
     - **Name**: `EurekaServer`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Netflix Eureka Server
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `EurekaServer`.

2. **Import the project into your IDE.**
   - Open your favorite IDE (e.g., IntelliJ, Eclipse, or VS Code).
   - Import the `EurekaServer` project as a Maven project.
   - Ensure that all dependencies are downloaded successfully.

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

4. **Configure the Eureka Server in `application.properties`.**
   - Open `src/main/resources/application.properties` and add the following:
     ```properties
     spring.application.name=eureka-server
     server.port=8761
     eureka.client.register-with-eureka=false
     eureka.client.fetch-registry=false
     ```

5. **Run the Eureka Server.**
   - Start the `EurekaServer` application using:
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
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Netflix Eureka Client
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `UserService`.

8. **Import the project into your IDE.**
   - Open your IDE and import the `UserService` project as a Maven project.
   - Verify that dependencies are downloaded correctly.

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

10. **Configure the Eureka client in `application.properties`.**
    - Add the following properties in the `UserService` project:
      ```properties
      spring.application.name=user-service
      server.port=8081
      eureka.client.service-url.defaultZone=http://localhost:8761/eureka
      ```

11. **Add a sample REST endpoint to the service.**
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

12. **Run the `UserService` application.**
    - Start the service using:
      ```bash
      ./mvnw spring-boot:run
      ```

13. **Verify service registration in Eureka.**
    - Refresh the Eureka dashboard at `http://localhost:8761`.
    - Confirm that the `user-service` is listed as a registered service.

---

### **Part 3: Adding Another Service**

14. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 7 and 8, but use the following configuration:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`

15. **Enable Eureka Client for `ProductService`.**
    - Add the `@EnableEurekaClient` annotation in the `ProductServiceApplication.java` file.

16. **Configure Eureka for `ProductService`.**
    - Add the following to the `application.properties` file:
      ```properties
      spring.application.name=product-service
      server.port=8082
      eureka.client.service-url.defaultZone=http://localhost:8761/eureka
      ```

17. **Add a REST endpoint for the `ProductService`.**
    - Create a new file `ProductController.java` in the `src/main/java/com/microservices/productservice` folder:
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
    - Start the service using:
      ```bash
      ./mvnw spring-boot:run
      ```

19. **Verify service registration in Eureka.**
    - Refresh the Eureka dashboard at `http://localhost:8761`.
    - Confirm that both `user-service` and `product-service` are listed as registered services.

---

### **Part 4: Testing Service Discovery**

20. **Test inter-service communication using Eureka.**
    - Use Postman or a browser to test the REST endpoints:
      - `UserService`: `http://localhost:8081/users`
      - `ProductService`: `http://localhost:8082/products`
    - Verify that both services respond correctly.

---

### **Optional Exercises (20 mins)**

1. **Add load balancing to `UserService`.**
   - Modify the `UserService` configuration to run multiple instances on different ports (e.g., `8081` and `8083`).
   - Verify that Eureka reflects the multiple instances.

2. **Integrate Ribbon for client-side load balancing.**
   - Add Ribbon dependency to a new client microservice and test load balancing between `UserService` instances.

# **Lab 15: Implement Contract Testing Using Spring Cloud Contract**

## **Objective**
Learn how to use Spring Cloud Contract to create and verify consumer-driven contracts between microservices. Implement contract testing between `UserService` (producer) and `OrderService` (consumer).

---

## **Lab Steps**

### **Part 1: Setting Up the Producer (UserService)**

1. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: `3.1.4`
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Contract Verifier (Version: `4.0.2`)
   - Extract the zip file into a folder named `UserService`.

2. **Import the `UserService` project into your IDE.**
   - Open your IDE and import the `UserService` project as a Maven project.

3. **Create a REST controller for `UserService`.**
   - Create a new file `UserController.java`:
     ```java
     package com.microservices.userservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.PathVariable;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class UserController {

         @GetMapping("/users/{id}")
         public User getUserById(@PathVariable String id) {
             return new User(id, "John Doe");
         }
     }

     class User {
         private String id;
         private String name;

         public User(String id, String name) {
             this.id = id;
             this.name = name;
         }

         public String getId() { return id; }
         public void setId(String id) { this.id = id; }
         public String getName() { return name; }
         public void setName(String name) { this.name = name; }
     }
     ```

4. **Add a contract file.**
   - Create a folder `src/test/resources/contracts` and add a file `user-get-contract.groovy`:
     ```groovy
     Contract.make {
         description "Should return user details for a given ID"
         request {
             method GET()
             urlPath('/users/1')
         }
         response {
             status 200
             body([
                 id: '1',
                 name: 'John Doe'
             ])
             headers {
                 contentType(applicationJson())
             }
         }
     }
     ```

5. **Enable Spring Cloud Contract Verifier.**
   - Add the following Maven plugin to `pom.xml`:
     ```xml
     <build>
         <plugins>
             <plugin>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-contract-maven-plugin</artifactId>
                 <version>4.0.2</version>
                 <extensions>true</extensions>
                 <configuration>
                     <baseClassForTests>com.microservices.userservice.BaseContractTest</baseClassForTests>
                 </configuration>
             </plugin>
         </plugins>
     </build>
     ```

6. **Create a base test class for the contract.**
   - Create a new file `BaseContractTest.java` in `src/test/java/com/microservices/userservice`:
     ```java
     package com.microservices.userservice;

     import io.restassured.module.mockmvc.RestAssuredMockMvc;
     import org.junit.jupiter.api.BeforeEach;

     public abstract class BaseContractTest {

         @BeforeEach
         void setUp() {
             RestAssuredMockMvc.standaloneSetup(new UserController());
         }
     }
     ```

7. **Generate stubs.**
   - Run the following Maven command to generate the stubs:
     ```bash
     ./mvnw clean install
     ```

8. **Verify the generated stubs.**
   - Ensure the stubs are available under `target/stubs`.

---

### **Part 2: Setting Up the Consumer (OrderService)**

9. **Generate a new Spring Boot project for `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: `3.1.4`
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Contract Stub Runner (Version: `4.0.2`)
       - Spring WebClient
   - Extract the zip file into a folder named `OrderService`.

10. **Import the `OrderService` project into your IDE.**
    - Open your IDE and import the `OrderService` project as a Maven project.

11. **Add stub dependency for `UserService`.**
    - Add the following to `pom.xml` to include the generated stubs:
      ```xml
      <dependency>
          <groupId>com.microservices</groupId>
          <artifactId>user-service</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <classifier>stubs</classifier>
          <scope>test</scope>
      </dependency>
      ```

12. **Configure Stub Runner for `OrderService`.**
    - Add the following properties in `src/test/resources/application.properties`:
      ```properties
      stubrunner.ids-to-fetch=com.microservices:user-service:+:stubs
      stubrunner.stubs-mode=LOCAL
      stubrunner.repository-root=target/stubs
      ```

13. **Create a REST controller in `OrderService`.**
    - Create a new file `OrderController.java`:
      ```java
      package com.microservices.orderservice;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;
      import org.springframework.web.reactive.function.client.WebClient;

      @RestController
      public class OrderController {

          @Autowired
          private WebClient.Builder webClientBuilder;

          @GetMapping("/orders")
          public String getOrders() {
              String user = webClientBuilder.build()
                      .get()
                      .uri("http://localhost:8081/users/1")
                      .retrieve()
                      .bodyToMono(String.class)
                      .block();

              return "Orders from OrderService and User: " + user;
          }
      }
      ```

14. **Add WebClient configuration.**
    - Add a `WebClient.Builder` bean in `OrderServiceApplication.java`:
      ```java
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.context.annotation.Bean;
      import org.springframework.web.reactive.function.client.WebClient;

      @SpringBootApplication
      public class OrderServiceApplication {

          public static void main(String[] args) {
              SpringApplication.run(OrderServiceApplication.class, args);
          }

          @Bean
          public WebClient.Builder webClientBuilder() {
              return WebClient.builder();
          }
      }
      ```

15. **Run `OrderService`.**
    - Start the application and test the `/orders` endpoint:
      ```bash
      ./mvnw spring-boot:run
      ```

---

### **Part 3: Testing the Contract**

16. **Create a contract test for `OrderService`.**
    - Create a new test file `OrderServiceContractTest.java` in `src/test/java/com/microservices/orderservice`:
      ```java
      package com.microservices.orderservice;

      import org.junit.jupiter.api.Test;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.cloud.contract.stubrunner.spring.AutoConfigureStubRunner;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.reactive.function.client.WebClient;

      @SpringBootTest
      @AutoConfigureStubRunner
      public class OrderServiceContractTest {

          @Autowired
          private WebClient.Builder webClientBuilder;

          @Test
          public void shouldFetchUserDetails() {
              String user = webClientBuilder.build()
                      .get()
                      .uri("http://localhost:8081/users/1")
                      .retrieve()
                      .bodyToMono(String.class)
                      .block();

              assert user.contains("John Doe");
          }
      }
      ```

17. **Run the contract test.**
    - Execute the tests using:
      ```bash
      ./mvnw test
      ```

18. **Verify test results.**
    - Confirm that the contract test passes, ensuring compatibility between `UserService` and `OrderService`.

19. **Push generated stubs to a repository (optional).**
    - Upload the generated stubs to a Git or artifact repository for other consumers to use.

20. **Validate the end-to-end setup.**
    - Call the `/orders` endpoint, generate new stubs, and rerun tests to confirm the integration.

---

### **Optional Exercises (20 mins)**

1. **Extend the contract.**
   - Add additional fields to the `User` object (e.g., `email`) and update the contract and tests.

2. **Add a third consumer.**
   - Create a `ProductService` and implement contract testing with `UserService`.

---

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

3. **Create a REST controller for `UserService`.**
   - Add `UserController.java`:
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
   - Create `src/test/resources/contracts/user-get-contract.groovy`:
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
   - Add `BaseContractTest.java`:
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
   - Run:
     ```bash
     ./mvnw clean install
     ```

8. **Verify the generated stubs.**
   - Confirm stubs are available under `target/stubs`.

---

### **Part 2: Setting Up the Consumer (OrderService)**

9. **Generate a new Spring Boot project for `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Contract Stub Runner
       - Spring WebClient
   - Extract the zip file into a folder named `OrderService`.

10. **Import the `OrderService` project into your IDE.**

11. **Add stub dependency for `UserService`.**
    - Add to `pom.xml`:
      ```xml
      <dependency>
          <groupId>com.microservices</groupId>
          <artifactId>user-service</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <classifier>stubs</classifier>
          <scope>test</scope>
      </dependency>
      ```

12. **Configure Stub Runner.**
    - Add to `src/test/resources/application.properties`:
      ```properties
      stubrunner.ids-to-fetch=com.microservices:user-service:+:stubs
      stubrunner.stubs-mode=LOCAL
      stubrunner.repository-root=target/stubs
      ```

13. **Create a REST controller in `OrderService`.**
    - Add `OrderController.java`:
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

14. **Run `OrderService`.**
    ```bash
    ./mvnw spring-boot:run
    ```

---

### **Part 3: Testing the Contract**

15. **Create a contract test for `OrderService`.**
    - Add `OrderServiceContractTest.java`:
      ```java
      package com.microservices.orderservice;

      import org.junit.jupiter.api.Test;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.cloud.contract.stubrunner.spring.AutoConfigureStubRunner;

      @SpringBootTest
      @AutoConfigureStubRunner
      public class OrderServiceContractTest {

          @Test
          public void shouldFetchUserDetails() {
              String user = WebClient.builder()
                      .baseUrl("http://localhost:8081")
                      .build()
                      .get()
                      .uri("/users/1")
                      .retrieve()
                      .bodyToMono(String.class)
                      .block();

              assert user.contains("John Doe");
          }
      }
      ```

16. **Run the tests.**
    ```bash
    ./mvnw test
    ```

---

### **Optional Exercises**

1. **Extend the contract.**
   - Add a new field (e.g., `email`) to the `User` object and update the contract.

2. **Add a third consumer.**
   - Create a `ProductService` and implement contract testing with `UserService`.

3. **Publish stubs to a central repository.**
   - Push generated stubs to an artifact repository like Nexus or Artifactory.

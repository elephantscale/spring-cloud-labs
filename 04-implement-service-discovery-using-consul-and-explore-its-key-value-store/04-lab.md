# **Lab 4: Implement Service Discovery Using Consul and Explore Its Key-Value Store**

## **Objective**
Set up a Consul server to enable dynamic service discovery and utilize its key-value (KV) store for managing configurations. Learn how to register microservices with Consul and query service information and KV pairs.

---

## **Lab Steps**

### **Part 1: Setting Up Consul**

1. **Download and install Consul.**
   - Visit [https://developer.hashicorp.com/consul/downloads](https://developer.hashicorp.com/consul/downloads).
   - Download the appropriate binary for your operating system, extract it, and add it to your system's PATH.

2. **Start Consul in development mode.**
   - Open a terminal and run:
     ```bash
     consul agent -dev
     ```
   - This starts Consul in development mode on `http://127.0.0.1:8500`.

3. **Access the Consul UI.**
   - Open a browser and navigate to:
     ```
     http://localhost:8500
     ```
   - Verify that the Consul dashboard is displayed, with no registered services yet.

4. **Add a key-value pair to the KV store.**
   - Navigate to the **Key/Value** tab in the Consul UI.
   - Click **Create** and add:
     - Key: `config/sample-service`
     - Value:
       ```json
       {
         "greeting": "Hello from Consul KV Store!"
       }
       ```
   - Save the key-value pair.

5. **Verify KV store retrieval using the CLI.**
   - Use the following command to fetch the KV pair:
     ```bash
     consul kv get config/sample-service
     ```
   - Confirm that the output matches the value you added in the UI.

---

### **Part 2: Setting Up the Consul Server in Spring Boot**

6. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `consul-server`
     - **Name**: `ConsulServer`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Consul Discovery
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `ConsulServer`.

7. **Import the project into your IDE.**
   - Open your IDE and import the `ConsulServer` project as a Maven project.

8. **Enable Consul Discovery.**
   - Open the `ConsulServerApplication.java` file in the `src/main/java/com/microservices/consulserver` folder.
   - Add the `@EnableDiscoveryClient` annotation:
     ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

     @SpringBootApplication
     @EnableDiscoveryClient
     public class ConsulServerApplication {
         public static void main(String[] args) {
             SpringApplication.run(ConsulServerApplication.class, args);
         }
     }
     ```

9. **Configure Consul in `application.properties`.**
   - Add the following properties in `src/main/resources/application.properties`:
     ```properties
     spring.application.name=consul-server
     spring.cloud.consul.host=127.0.0.1
     spring.cloud.consul.port=8500
     server.port=8081
     ```

10. **Add a sample REST endpoint.**
    - Create a new file `ConsulController.java` in the `src/main/java/com/microservices/consulserver` folder:
      ```java
      package com.microservices.consulserver;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ConsulController {

          @GetMapping("/greeting")
          public String greeting() {
              return "Hello from Consul Server!";
          }
      }
      ```

11. **Run the `ConsulServer` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

12. **Verify service registration in Consul.**
    - Open the Consul UI at `http://localhost:8500`.
    - Navigate to the **Services** tab and confirm that `consul-server` is listed.

---

### **Part 3: Adding Another Service**

13. **Generate a new Spring Boot project for the `UserService`.**
    - Repeat steps 6 and 7, but use the following configuration:
      - **Artifact Id**: `user-service`
      - **Name**: `UserService`

14. **Enable Consul Discovery for `UserService`.**
    - Add the `@EnableDiscoveryClient` annotation in the `UserServiceApplication.java` file.

15. **Configure Consul for `UserService`.**
    - Add the following to the `application.properties` file:
      ```properties
      spring.application.name=user-service
      spring.cloud.consul.host=127.0.0.1
      spring.cloud.consul.port=8500
      server.port=8082
      ```

16. **Add a REST endpoint for the `UserService`.**
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

17. **Run the `UserService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

18. **Verify service registration in Consul.**
    - Open the Consul UI and confirm that both `consul-server` and `user-service` are listed.

---

### **Part 4: Exploring KV Store in Spring Boot**

19. **Fetch KV store values dynamically in the `ConsulServer`.**
    - Update `ConsulController` to retrieve values from the KV store:
      ```java
      package com.microservices.consulserver;

      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ConsulController {

          @Value("${greeting:Default Greeting}")
          private String greeting;

          @GetMapping("/greeting")
          public String getGreeting() {
              return greeting;
          }
      }
      ```

20. **Test KV store retrieval.**
    - Open a browser or use Postman to access:
      ```
      http://localhost:8081/greeting
      ```
    - Confirm that the response matches the KV store value (`"Hello from Consul KV Store!"`).

---

### **Optional Exercises (20 mins)**

1. **Add environment-specific KV store keys.**
   - Create keys like `config/sample-service/dev` and `config/sample-service/prod`.
   - Modify `spring.profiles.active=dev` or `prod` in the application and test KV retrieval.

2. **Simulate a service failure in Consul.**
   - Stop one of the services (e.g., `UserService`) and verify how Consul handles service health checks.

---

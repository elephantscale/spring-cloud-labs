# **Lab 4: Implement Service Discovery Using Consul and Explore Its Key-Value Store**

## **Objective**
Set up a Consul server to enable dynamic service discovery and utilize its key-value (KV) store for managing configurations. Learn how to register microservices with Consul and query service information and KV pairs.

---

## **Lab Steps**

### **Part 1: Installing and Running Consul**

1. **Download the Consul binary.**
   - Visit [https://developer.hashicorp.com/consul/downloads](https://developer.hashicorp.com/consul/downloads) and download the latest Consul binary for your operating system.

2. **Extract the Consul binary.**
   - **Windows**:
     - Unzip the downloaded file (e.g., `consul_1.15.1_windows_amd64.zip`) to a folder, such as `C:\HashiCorp\Consul`.
   - **macOS/Linux**:
     - Extract the file to a directory, e.g., `/usr/local/bin/consul`:
       ```bash
       unzip consul_1.15.1_<platform>.zip -d /usr/local/bin/
       ```

3. **Add Consul to the system PATH.**
   - **Windows**:
     - Add the folder containing `consul.exe` to the PATH via Environment Variables.
   - **macOS/Linux**:
     - Add Consul to the PATH:
       ```bash
       export PATH=$PATH:/usr/local/bin/consul
       ```
     - Reload the configuration:
       ```bash
       source ~/.bashrc
       ```

4. **Start Consul in development mode.**
   - Open a terminal and run:
     ```bash
     consul agent -dev
     ```
   - Consul will start in development mode and be accessible at `http://127.0.0.1:8500`.

5. **Access the Consul UI.**
   - Open a browser and navigate to:
     ```
     http://127.0.0.1:8500
     ```
   - Verify that the Consul dashboard is displayed with no registered services yet.

6. **Add a key-value pair to the KV store.**
   - Navigate to the **Key/Value** tab in the Consul UI.
   - Click **Create** and add:
     - **Key**: `config/sample-service`
     - **Value**:
       ```json
       {
         "greeting": "Hello from Consul KV Store!"
       }
       ```
   - Save the key-value pair.

7. **Verify KV store retrieval using the CLI.**
   - Run the following command:
     ```bash
     consul kv get config/sample-service
     ```
   - Confirm that the output matches the value you added in the UI.

---

### **Part 2: Setting Up the Consul Server in Spring Boot**

8. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `consul-server`
     - **Name**: `ConsulServer`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Consul Discovery
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `ConsulServer`.

9. **Import the project into your IDE.**

10. **Enable Consul Discovery.**
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

11. **Configure Consul in `application.yml`.**
    - Create a file named `application.yml` in the `src/main/resources` folder and add:
      ```yaml
      spring:
        application:
          name: consul-server
        cloud:
          consul:
            host: 127.0.0.1
            port: 8500

      server:
        port: 8081
      ```

12. **Add a sample REST endpoint.**
    - Create `ConsulController.java` in the `src/main/java/com/microservices/consulserver` folder:
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

13. **Run the `ConsulServer` application.**
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

14. **Verify service registration in Consul.**
    - Open the Consul UI at `http://127.0.0.1:8500`.
    - Navigate to the **Services** tab and confirm that `consul-server` is listed.

---

### **Part 3: Adding Another Service**

15. **Generate a new Spring Boot project for the `UserService`.**
    - Repeat steps 8 and 9, but use:
      - **Artifact Id**: `user-service`
      - **Name**: `UserService`.

16. **Enable Consul Discovery for `UserService`.**
    - Add the `@EnableDiscoveryClient` annotation in the `UserServiceApplication.java` file.

17. **Configure Consul for `UserService`.**
    - Create `application.yml` in the `src/main/resources` folder and add:
      ```yaml
      spring:
        application:
          name: user-service
        cloud:
          consul:
            host: 127.0.0.1
            port: 8500

      server:
        port: 8082
      ```

18. **Add a REST endpoint for the `UserService`.**
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

19. **Run the `UserService` application.**
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

20. **Verify service registration in Consul.**
    - Open the Consul UI and confirm that both `consul-server` and `user-service` are listed.

---

### **Part 4: Exploring KV Store in Spring Boot**

21. **Fetch KV store values dynamically in the `ConsulServer`.**
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

22. **Test KV store retrieval.**
    - Open a browser or use Postman to access:
      ```
      http://localhost:8081/greeting
      ```
    - Confirm that the response matches the KV store value (`"Hello from Consul KV Store!"`).

---

### **Optional Exercises**

1. **Add environment-specific KV store keys.**
   - Create keys like `config/sample-service/dev` and `config/sample-service/prod`.
   - Modify `spring.profiles.active=dev` or `prod` in the application and test KV retrieval.

2. **Simulate a service failure in Consul.**
   - Stop one of the services (e.g., `UserService`) and verify how Consul handles service health checks.

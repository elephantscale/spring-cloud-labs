# **Lab 5: Use Zookeeper for Distributed Configuration and Synchronization**

## **Objective**
Set up Apache Zookeeper as a configuration and synchronization server. Learn how to store and retrieve configurations, register microservices, and explore Zookeeper's capabilities for distributed systems.

---

## **Lab Steps**

### **Part 1: Installing and Running Zookeeper on Windows**

1. **Download Zookeeper.**
   - Visit [https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html) and download the latest stable release (e.g., `zookeeper-3.8.2`).

2. **Extract the downloaded archive.**
   - Extract the downloaded file (e.g., `apache-zookeeper-3.8.2-bin.tar.gz`) to a folder such as `C:\Zookeeper`.

3. **Set up the Zookeeper configuration.**
   - Navigate to the `conf` folder:
     ```
     cd C:\Zookeeper\conf
     ```
   - Copy `zoo_sample.cfg` and rename it to `zoo.cfg`:
     ```
     copy zoo_sample.cfg zoo.cfg
     ```

4. **Update the configuration file.**
   - Open `zoo.cfg` in a text editor (e.g., Notepad or VS Code).
   - Ensure the `dataDir` is set to a valid directory, such as:
     ```properties
     dataDir=C:/Zookeeper/data
     ```
   - Create the `data` directory:
     ```
     mkdir C:\Zookeeper\data
     ```

5. **Start the Zookeeper server.**
   - Open a Command Prompt, navigate to the Zookeeper folder, and run:
     ```
     bin\zkServer.cmd
     ```
   - Zookeeper will start, and the server logs will appear in the terminal.

6. **Verify Zookeeper is running.**
   - Check the server's status using:
     ```
     bin\zkServer.cmd status
     ```
   - Confirm that the server is running and in standalone mode.

7. **Test Zookeeper using the CLI.**
   - Open a Zookeeper shell by running:
     ```
     bin\zkCli.cmd
     ```
   - Create a test node:
     ```
     create /config "Hello Zookeeper"
     ```
   - Retrieve the node value:
     ```
     get /config
     ```
   - Confirm that the value `Hello Zookeeper` is displayed.

---

### **Part 2: Setting Up the Spring Boot Application**

8. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `zookeeper-service`
     - **Name**: `ZookeeperService`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Zookeeper Discovery
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `ZookeeperService`.

9. **Import the project into your IDE.**
    - Open your IDE and import the `ZookeeperService` project as a Maven project.

10. **Enable Zookeeper Discovery.**
    - Open the `ZookeeperServiceApplication.java` file in the `src/main/java/com/microservices/zookeeperservice` folder.
    - Add the `@EnableDiscoveryClient` annotation:
      ```java
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

      @SpringBootApplication
      @EnableDiscoveryClient
      public class ZookeeperServiceApplication {
          public static void main(String[] args) {
              SpringApplication.run(ZookeeperServiceApplication.class, args);
          }
      }
      ```

11. **Configure Zookeeper in `application.properties`.**
    - Add the following configurations:
      ```properties
      spring.application.name=zookeeper-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8083
      ```

12. **Add a REST endpoint to the service.**
    - Create a new file `ZookeeperController.java` in the `src/main/java/com/microservices/zookeeperservice` folder:
      ```java
      package com.microservices.zookeeperservice;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ZookeeperController {

          @GetMapping("/zookeeper")
          public String getMessage() {
              return "Hello from Zookeeper Service!";
          }
      }
      ```

13. **Run the `ZookeeperService` application.**
    - Start the application using:
      ```
      mvnw spring-boot:run
      ```

14. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI (`bin\zkCli.cmd`) and run:
      ```
      ls /services
      ```
    - Confirm that `zookeeper-service` is listed.

---

### **Part 3: Adding Another Service**

15. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 8 and 9, but use the following configuration:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`.

16. **Enable Zookeeper Discovery for `ProductService`.**
    - Add the `@EnableDiscoveryClient` annotation in the `ProductServiceApplication.java` file.

17. **Configure Zookeeper for `ProductService`.**
    - Add the following to the `application.properties` file:
      ```properties
      spring.application.name=product-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8084
      ```

18. **Add a REST endpoint for the `ProductService`.**
    - Create a new file `ProductController.java` in the `src/main/java/com/microservices/productservice` folder:
      ```java
      package com.microservices.productservice;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ProductController {

          @GetMapping("/products")
          public String getProducts() {
              return "List of products from Zookeeper!";
          }
      }
      ```

19. **Run the `ProductService` application.**
    - Start the application using:
      ```
      mvnw spring-boot:run
      ```

20. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI and run:
      ```
      ls /services
      ```
    - Confirm that both `zookeeper-service` and `product-service` are listed.

---

### **Part 4: Testing Configuration Management**

21. **Update Zookeeper’s KV data.**
    - In the Zookeeper CLI, update the value of the `/config` node:
      ```
      set /config "Updated Configuration Value"
      ```

22. **Retrieve the updated configuration in `ZookeeperService`.**
    - Modify the `ZookeeperController` to dynamically fetch the value:
      ```java
      import org.springframework.beans.factory.annotation.Value;

      @RestController
      public class ZookeeperController {

          @Value("${config:Default Config}")
          private String configValue;

          @GetMapping("/config")
          public String getConfig() {
              return configValue;
          }
      }
      ```

23. **Test the `/config` endpoint.**
    - Open a browser or use Postman to access:
      ```
      http://localhost:8083/config
      ```
    - Verify that the updated configuration value is displayed.

---

### **Optional Exercises (20 mins)**

1. **Simulate service failures.**
   - Stop one of the services (e.g., `ProductService`) and verify how Zookeeper handles deregistration.

2. **Add environment-specific configurations.**
   - Store configurations for multiple environments (`/config/dev` and `/config/prod`) and test retrieval based on active profiles.

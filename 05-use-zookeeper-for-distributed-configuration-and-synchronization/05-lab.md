# **Lab 5: Use Zookeeper for Distributed Configuration and Synchronization**

## **Objective**
Set up Apache Zookeeper as a configuration and synchronization server. Learn how to store and retrieve configurations, register microservices, and explore Zookeeper's capabilities for distributed systems.

---

## **Lab Steps**

### **Part 1: Setting Up Zookeeper**

1. **Download and install Zookeeper.**
   - Visit [https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html).
   - Download the latest stable release (e.g., `zookeeper-3.8.x`) for your operating system.
   - Extract the downloaded archive to a directory of your choice.

2. **Configure Zookeeper.**
   - Navigate to the `conf` directory of the extracted Zookeeper folder.
   - Rename `zoo_sample.cfg` to `zoo.cfg`.

3. **Start the Zookeeper server.**
   - Open a terminal, navigate to the Zookeeper folder, and run:
     ```bash
     bin/zkServer.sh start
     ```
   - Zookeeper will start on `localhost:2181`.

4. **Verify Zookeeper is running.**
   - Use the following command to check the server's status:
     ```bash
     bin/zkServer.sh status
     ```
   - Confirm the server is running and in standalone mode.

5. **Test Zookeeper using the CLI.**
   - Open a Zookeeper shell by running:
     ```bash
     bin/zkCli.sh
     ```
   - Create a test node:
     ```bash
     create /config "Hello Zookeeper"
     ```
   - Retrieve the node value:
     ```bash
     get /config
     ```
   - Confirm that the value `Hello Zookeeper` is displayed.

---

### **Part 2: Setting Up the Spring Boot Application**

6. **Generate a new Spring Boot project using Spring Initializr.**
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

7. **Import the project into your IDE.**
   - Open your IDE and import the `ZookeeperService` project as a Maven project.

8. **Enable Zookeeper Discovery.**
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

9. **Configure Zookeeper in `application.properties`.**
   - Add the following configurations:
     ```properties
     spring.application.name=zookeeper-service
     spring.cloud.zookeeper.connect-string=localhost:2181
     server.port=8083
     ```

10. **Add a REST endpoint to the service.**
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

11. **Run the `ZookeeperService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

12. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI (`bin/zkCli.sh`) and run:
      ```bash
      ls /services
      ```
    - Confirm that `zookeeper-service` is listed.

---

### **Part 3: Adding Another Service**

13. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 6 and 7, but use the following configuration:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`

14. **Enable Zookeeper Discovery for `ProductService`.**
    - Add the `@EnableDiscoveryClient` annotation in the `ProductServiceApplication.java` file.

15. **Configure Zookeeper for `ProductService`.**
    - Add the following to the `application.properties` file:
      ```properties
      spring.application.name=product-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8084
      ```

16. **Add a REST endpoint for the `ProductService`.**
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

17. **Run the `ProductService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

18. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI and run:
      ```bash
      ls /services
      ```
    - Confirm that both `zookeeper-service` and `product-service` are listed.

---

### **Part 4: Testing Configuration Management**

19. **Update Zookeeper’s KV data.**
    - In the Zookeeper CLI, update the value of the `/config` node:
      ```bash
      set /config "Updated Configuration Value"
      ```

20. **Retrieve the updated configuration in `ZookeeperService`.**
    - Modify the `ZookeeperController` to dynamically fetch the value:
      ```java
      @Value("${config:Default Config}")
      private String configValue;

      @GetMapping("/config")
      public String getConfig() {
          return configValue;
      }
      ```

21. **Test the `/config` endpoint.**
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

---

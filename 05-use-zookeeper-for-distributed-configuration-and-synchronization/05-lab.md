# **Lab 5: Use Zookeeper for Distributed Configuration and Synchronization**

## **Objective**
Set up Apache Zookeeper as a configuration and synchronization server. Learn how to store and retrieve configurations, register microservices, and explore Zookeeper's capabilities for distributed systems.

---

## **Lab Steps**

### **Part 1: Installing and Running Zookeeper**

1. **Download Zookeeper.**
   - Visit [https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html) and download the latest stable release (e.g., `zookeeper-3.8.2`).

2. **Extract the downloaded archive.**
   - **Windows**:
     - Extract the file (e.g., `apache-zookeeper-3.8.2-bin.tar.gz`) to a folder such as `C:\Zookeeper`.
   - **macOS/Linux**:
     - Extract the file to a suitable directory:
       ```bash
       tar -xvzf apache-zookeeper-3.8.2-bin.tar.gz -C ~/Zookeeper
       ```

3. **Set up the Zookeeper configuration.**
   - Navigate to the `conf` folder:
     ```bash
     cd [Zookeeper-extracted-folder]/conf
     ```
   - Copy `zoo_sample.cfg` and rename it to `zoo.cfg`:
     ```bash
     cp zoo_sample.cfg zoo.cfg
     ```

4. **Update the configuration file.**
   - Open `zoo.cfg` in a text editor.
   - Ensure the `dataDir` points to a valid directory:
     ```properties
     dataDir=/path/to/Zookeeper/data
     ```
   - Create the `data` directory:
     ```bash
     mkdir -p /path/to/Zookeeper/data
     ```

5. **Start the Zookeeper server.**
   - Navigate to the Zookeeper folder and run:
     ```bash
     bin/zkServer.sh start
     ```

6. **Verify Zookeeper is running.**
   - Check the server's status:
     ```bash
     bin/zkServer.sh status
     ```
   - Confirm that the server is running and in standalone mode.

7. **Test Zookeeper using the CLI.**
   - Open the Zookeeper shell:
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

8. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
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

10. **Enable Zookeeper Discovery.**
    - Open `ZookeeperServiceApplication.java` in the `src/main/java/com/microservices/zookeeperservice` folder.
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

11. **Configure Zookeeper in `application.yml`.**
    - Create `application.yml` in `src/main/resources` and add:
      ```yaml
      spring:
        application:
          name: zookeeper-service
        cloud:
          zookeeper:
            connect-string: localhost:2181

      server:
        port: 8083
      ```

12. **Add a REST endpoint to the service.**
    - Create `ZookeeperController.java` in the `src/main/java/com/microservices/zookeeperservice` folder:
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
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

14. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI and run:
      ```bash
      ls /services
      ```
    - Confirm that `zookeeper-service` is listed.

---

### **Part 3: Adding Another Service**

15. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 8 and 9, using the following configuration:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`.

16. **Enable Zookeeper Discovery for `ProductService`.**
    - Add the `@EnableDiscoveryClient` annotation in `ProductServiceApplication.java`.

17. **Configure Zookeeper for `ProductService`.**
    - Create `application.yml` in `src/main/resources` and add:
      ```yaml
      spring:
        application:
          name: product-service
        cloud:
          zookeeper:
            connect-string: localhost:2181

      server:
        port: 8084
      ```

18. **Add a REST endpoint for `ProductService`.**
    - Create `ProductController.java` in `src/main/java/com/microservices/productservice`:
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
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

20. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI and run:
      ```bash
      ls /services
      ```
    - Confirm that both `zookeeper-service` and `product-service` are listed.

---

### **Part 4: Testing Configuration Management**

21. **Update Zookeeper’s KV data.**
    - In the Zookeeper CLI, update the value of the `/config` node:
      ```bash
      set /config "Updated Configuration Value"
      ```

22. **Retrieve the updated configuration in `ZookeeperService`.**
    - Modify the `ZookeeperController` to dynamically fetch the value:
      ```java
      package com.microservices.zookeeperservice;

      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

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

### **Optional Exercises**

1. **Simulate service failures.**
   - Stop one of the services (e.g., `ProductService`) and verify how Zookeeper handles deregistration.

2. **Add environment-specific configurations.**
   - Store configurations for multiple environments (`/config/dev` and `/config/prod`) and test retrieval based on active profiles.

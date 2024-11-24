# **Lab 5: Use Zookeeper for Distributed Configuration and Synchronization**

## **Objective**
Set up Apache Zookeeper as a configuration and synchronization server. Learn how to store and retrieve configurations, register microservices, and explore Zookeeper's capabilities for distributed systems.

---

## **Lab Steps**

### **Part 1: Installing and Running Zookeeper on Linux**

1. **Update Linux packages.**
   - Open a terminal and run:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

2. **Download Zookeeper.**
   - Visit [https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html) and copy the link for the latest stable release.
   - Use `wget` to download Zookeeper:
     ```bash
     wget https://downloads.apache.org/zookeeper/zookeeper-3.8.2/apache-zookeeper-3.8.2-bin.tar.gz
     ```

3. **Extract the downloaded archive.**
   - Run the following command to extract the Zookeeper archive:
     ```bash
     tar -xvzf apache-zookeeper-3.8.2-bin.tar.gz
     ```
   - Move the extracted folder to `/opt`:
     ```bash
     sudo mv apache-zookeeper-3.8.2-bin /opt/zookeeper
     ```

4. **Set up Zookeeper configuration.**
   - Navigate to the `conf` directory:
     ```bash
     cd /opt/zookeeper/conf
     ```
   - Rename `zoo_sample.cfg` to `zoo.cfg`:
     ```bash
     mv zoo_sample.cfg zoo.cfg
     ```

5. **Update the configuration file.**
   - Edit the `zoo.cfg` file and ensure the `dataDir` path is set:
     ```properties
     dataDir=/opt/zookeeper/data
     ```
   - Create the `data` directory:
     ```bash
     mkdir /opt/zookeeper/data
     ```

6. **Start the Zookeeper server.**
   - Navigate to the Zookeeper home directory:
     ```bash
     cd /opt/zookeeper
     ```
   - Run the following command to start Zookeeper:
     ```bash
     bin/zkServer.sh start
     ```

7. **Verify Zookeeper is running.**
   - Check the server's status using:
     ```bash
     bin/zkServer.sh status
     ```
   - Confirm that the server is running and in standalone mode.

8. **Test Zookeeper using the CLI.**
   - Open a Zookeeper shell:
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

9. **Generate a new Spring Boot project using Spring Initializr.**
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

10. **Import the project into your IDE.**
    - Open your IDE and import the `ZookeeperService` project as a Maven project.

11. **Enable Zookeeper Discovery.**
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

12. **Configure Zookeeper in `application.properties`.**
    - Add the following configurations:
      ```properties
      spring.application.name=zookeeper-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8083
      ```

13. **Add a REST endpoint to the service.**
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

14. **Run the `ZookeeperService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

15. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI (`bin/zkCli.sh`) and run:
      ```bash
      ls /services
      ```
    - Confirm that `zookeeper-service` is listed.

---

### **Part 3: Adding Another Service**

16. **Generate a new Spring Boot project for the `ProductService`.**
    - Repeat steps 9 and 10, but use the following configuration:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`

17. **Enable Zookeeper Discovery for `ProductService`.**
    - Add the `@EnableDiscoveryClient` annotation in the `ProductServiceApplication.java` file.

18. **Configure Zookeeper for `ProductService`.**
    - Add the following to the `application.properties` file:
      ```properties
      spring.application.name=product-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8084
      ```

19. **Add a REST endpoint for the `ProductService`.**
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

20. **Run the `ProductService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

21. **Verify service registration in Zookeeper.**
    - Open the Zookeeper CLI and run:
      ```bash
      ls /services
      ```
    - Confirm that both `zookeeper-service` and `product-service` are listed.

---

### **Part 4: Testing Configuration Management**

22. **Update Zookeeper’s KV data.**
    - In the Zookeeper CLI, update the value of the `/config` node:
      ```bash
      set /config "Updated Configuration Value"
      ```

23. **Retrieve the updated configuration in `ZookeeperService`.**
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

24. **Test the `/config` endpoint.**
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


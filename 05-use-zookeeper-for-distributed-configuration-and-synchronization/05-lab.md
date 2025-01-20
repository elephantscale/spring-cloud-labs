# **Lab 5: Use Zookeeper for Distributed Configuration and Synchronization (Spring Boot 3.4.1)**

## **Objective**
Learn how to set up **Apache Zookeeper** to register microservices (Spring Boot **3.4.1**) for discovery and explore its **KV store** for dynamic configuration. Zookeeper offers distributed synchronization capabilities suitable for microservices needing coordination or shared config.

---

## **Lab Steps**

### **Part 1: Installing and Running Zookeeper**

1. **Download Zookeeper.**
   - Visit [Zookeeper Releases](https://zookeeper.apache.org/releases.html) and download the latest stable release (e.g., `zookeeper-3.8.2`).

2. **Extract the Zookeeper archive.**
   - **Windows**:
     - Extract (e.g., `apache-zookeeper-3.8.2-bin.tar.gz`) into `C:\Zookeeper`.
   - **macOS/Linux**:
     - Extract:
       ```bash
       tar -xvzf apache-zookeeper-3.8.2-bin.tar.gz -C ~/Zookeeper
       ```

3. **Set up the Zookeeper configuration.**
   - In the `conf` folder:
     ```bash
     cp zoo_sample.cfg zoo.cfg
     ```
   - Edit `zoo.cfg` to set a valid data directory:
     ```properties
     dataDir=/path/to/Zookeeper/data
     ```
   - Create that directory:
     ```bash
     mkdir -p /path/to/Zookeeper/data
     ```

4. **Start the Zookeeper server.**
   - **macOS/Linux** example:
     ```bash
     cd ~/Zookeeper/apache-zookeeper-3.8.2-bin
     bin/zkServer.sh start
     ```
   - **Windows** (in Powershell or Command Prompt):
     ```ps1
     cd C:\Zookeeper\apache-zookeeper-3.8.2-bin
     .\bin\zkServer.cmd
     ```

5. **Verify Zookeeper is running.**
   - Check status:
     ```bash
     bin/zkServer.sh status
     ```
   - It should say "standalone mode" and "running" if successful.

6. **Test Zookeeper using the CLI.**
   - Start the ZK shell:
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
   - Confirm it shows `Hello Zookeeper`.

---

### **Part 2: Setting Up the Zookeeper-Enabled Microservice**

7. **Generate a new Spring Boot project for `ZookeeperService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `zookeeper-service`
     - **Name**: `ZookeeperService`
     - **Package Name**: `com.microservices.zookeeperservice`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Zookeeper Discovery
       - Spring Boot Actuator
   - Extract into `ZookeeperService`.

8. **Import the project into your IDE.**

9. **Enable Zookeeper Discovery.**
   - In `ZookeeperServiceApplication.java`:
     ```java
     package com.microservices.zookeeperservice;

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

10. **Configure Zookeeper in `application.properties`.**
    - In `src/main/resources/application.properties`:
      ```properties
      spring.application.name=zookeeper-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8083
      ```
    - Port **2181** is the default Zookeeper port.

11. **Add a REST endpoint.**
    - In `src/main/java/com/microservices/zookeeperservice/ZookeeperController.java`:
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

12. **Run the `ZookeeperService`.**
    - From the `ZookeeperService` directory:
      ```bash
      ./mvnw spring-boot:run
      ```
    - If everything works, Zookeeper logs should show registration of **zookeeper-service**.

13. **Verify service registration.**
    - Open another terminal:
      ```bash
      bin/zkCli.sh
      ```
    - Run:
      ```bash
      ls /services
      ```
    - You should see `zookeeper-service`.

---

### **Part 3: Adding Another Service**

14. **Generate a new Spring Boot project: `ProductService`.**
    - Same approach:
      - **Artifact Id**: `product-service`
      - **Name**: `ProductService`
      - **Package Name**: `com.microservices.productservice`
      - **Dependencies**: 
        - Spring Web
        - Spring Cloud Zookeeper Discovery
        - Spring Boot Actuator
    - Extract into `ProductService`.

15. **Enable Zookeeper Discovery.**
    - In `ProductServiceApplication.java`:
      ```java
      package com.microservices.productservice;

      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

      @SpringBootApplication
      @EnableDiscoveryClient
      public class ProductServiceApplication {
          public static void main(String[] args) {
              SpringApplication.run(ProductServiceApplication.class, args);
          }
      }
      ```

16. **Configure Zookeeper for `ProductService`.**
    - In `src/main/resources/application.properties`:
      ```properties
      spring.application.name=product-service
      spring.cloud.zookeeper.connect-string=localhost:2181
      server.port=8084
      ```

17. **Add a REST endpoint for `ProductService`.**
    - In `ProductController.java`:
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

18. **Run `ProductService`.**
    - Similar to before:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check logs for any registration errors.

19. **Verify `ProductService` in Zookeeper.**
    - In the ZK CLI:
      ```bash
      ls /services
      ```
    - Should now list `zookeeper-service` and `product-service`.

---

### **Part 4: Testing Configuration Management**

20. **Update Zookeeper’s KV data.**
    - In the ZK CLI:
      ```bash
      set /config "Updated Configuration Value"
      ```
    - This changes the node’s data at path `/config`.

21. **Retrieve the updated configuration in `ZookeeperService`.**
    - Modify `ZookeeperController`:
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

22. **Test the `/config` endpoint.**
    - Access:
      ```
      http://localhost:8083/config
      ```
    - Should display **Updated Configuration Value** if `/config` node is read as a property.

---

## **Optional Exercises**

1. **Simulate service failures.**
   - Kill one service. ZK should reflect it in the ephemeral node structure.

2. **Add environment-specific configurations.**
   - E.g., `/config/dev` vs. `/config/prod`. Use profiles or direct property binding to read them.

3. **Integrate watchers.**
   - Zookeeper watchers let you get notified of config changes in real-time.

---

## **Conclusion**
By completing this lab, you have:
- **Installed and ran** Zookeeper locally.
- **Registered** multiple Spring Boot 3.4.1 services using **Spring Cloud Zookeeper Discovery**.
- **Explored the KV store** to set and retrieve dynamic configurations, which microservices can consume at runtime.
- Gained an understanding of how Zookeeper can provide **service discovery and configuration management**, similar to Eureka or Consul, but with additional synchronization primitives.

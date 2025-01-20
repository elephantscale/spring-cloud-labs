# **Lab 9: Set Up Event-Based Messaging Between Microservices Using Spring Cloud Bus (Spring Boot 3.4.1)**

## **Objective**
Learn how to use **Spring Cloud Bus** with **Spring Boot 3.4.1** to propagate configuration changes (and other events) among microservices in real time. By integrating **Spring Cloud Config Server**, **Kafka**, and **Spring Cloud Bus**, you’ll enable multiple microservices to automatically refresh updates without manual restarts.

---

## **Lab Steps**

### **Part 1: Setting Up the Configuration Server**

1. **Generate a new Spring Boot project for `ConfigServer`.**
   - Go to [https://start.spring.io/](https://start.spring.io/).
   - Configure:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-server`
     - **Name**: `ConfigServer`
     - **Dependencies**:
       - Spring Cloud Config Server
       - Spring Boot Actuator
       - Spring Cloud Bus
       - Spring Cloud Stream Kafka
   - Extract into a folder named `ConfigServer`.

2. **Import the project into your IDE.**

3. **Enable Spring Cloud Config Server.**
   - In `ConfigServerApplication.java`:
     ```java
     package com.microservices.configserver;

     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.config.server.EnableConfigServer;

     @SpringBootApplication
     @EnableConfigServer
     public class ConfigServerApplication {
         public static void main(String[] args) {
             SpringApplication.run(ConfigServerApplication.class, args);
         }
     }
     ```

4. **Configure the Config Server in `application.properties`.**
   - In `src/main/resources/application.properties`, add:
     ```properties
     server.port=8888
     spring.application.name=config-server

     spring.cloud.config.server.git.uri=https://github.com/<your-username>/config-repo
     spring.cloud.bus.enabled=true
     spring.cloud.stream.kafka.binder.brokers=localhost:9092
     ```
   - Replace `<your-username>` with your GitHub name if needed.

5. **Run the Config Server.**
   - From `ConfigServer`:
     ```bash
     ./mvnw spring-boot:run
     ```
   - The server starts on port **8888**.

---

### **Part 2: Setting Up the Kafka Broker**

6. **Download and install Apache Kafka.**
   - Visit [Apache Kafka Downloads](https://kafka.apache.org/downloads) to get the latest version.
   - Extract it (e.g., to `/opt/kafka`).

7. **Start Zookeeper.**
   - In one terminal:
     ```bash
     cd /opt/kafka
     ./bin/zookeeper-server-start.sh config/zookeeper.properties
     ```

8. **Start Kafka.**
   - In another terminal:
     ```bash
     ./bin/kafka-server-start.sh config/server.properties
     ```

9. **Verify Kafka is running.**
   - Use:
     ```bash
     ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
     ```
   - If no topics are listed, Kafka is running but with no custom topics yet.

---

### **Part 3: Setting Up a Client Service (`UserService`)**

10. **Generate a new Spring Boot project for `UserService`.**
    - Configure:
      - **Artifact Id**: `user-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Config Client
        - Spring Cloud Bus
        - Spring Cloud Stream Kafka
    - Extract into `UserService`.

11. **Import the project into your IDE.**

12. **Configure the `UserService` to use the Config Server.**
    - In `src/main/resources/application.properties`:
      ```properties
      server.port=8081
      spring.application.name=user-service

      spring.cloud.config.uri=http://localhost:8888
      spring.cloud.bus.enabled=true
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      ```
    - The application name `user-service` helps the config server locate the right properties file.

13. **Create a REST controller to display configuration values.**
    - In `src/main/java/com/microservices/userservice/ConfigController.java`:
      ```java
      package com.microservices.userservice;

      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ConfigController {

          @Value("${message:Default message}")
          private String message;

          @GetMapping("/message")
          public String getMessage() {
              return message;
          }
      }
      ```

14. **Run `UserService`.**
    - From the `UserService` folder:
      ```bash
      ./mvnw spring-boot:run
      ```

15. **Verify configuration retrieval.**
    - In a browser:
      ```
      http://localhost:8081/message
      ```
    - It should show the `message` property from your **Git** config repo (e.g., `application.yml` or `user-service.yml`).

---

### **Part 4: Testing Event-Based Messaging**

16. **Update the configuration file in the Git repo.**
    - For instance, if you have `message=Hello from Config Server!` in `user-service.properties`, change it to:
      ```properties
      message=Updated message from Config Server
      ```
    - Commit and push:
      ```bash
      git add .
      git commit -m "Updated message property"
      git push origin main
      ```

17. **Trigger a refresh event.**
    - Post a request to the **Config Server**’s `/actuator/bus-refresh` endpoint:
      ```bash
      curl -X POST http://localhost:8888/actuator/bus-refresh
      ```
    - This publishes an event over **Spring Cloud Bus** via **Kafka**.

18. **Verify the updated config in `UserService`.**
    - Reload:
      ```
      http://localhost:8081/message
      ```
    - Confirm the new `"Updated message from Config Server"` is displayed without restarting `UserService`.

---

### **Part 5: Adding Another Client Service (`ProductService`)**

19. **Generate a new project for `ProductService`.**
    - Similar to `UserService`:
      - **Artifact Id**: `product-service`
      - **Server port**: `8082`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Config Client
        - Spring Cloud Bus
        - Spring Cloud Stream Kafka

20. **Check config sync between services.**
    - After configuring `ProductService` similarly (application name `product-service`), trigger `/actuator/bus-refresh` again.
    - Both services (`UserService` and `ProductService`) should reflect updated properties from your Git config repo.

---

## **Optional Exercises**

1. **Add a new property to the configuration.**
   - e.g., `service.version=1.0`. Test that it appears in both services after a bus refresh.

2. **Manually test Kafka messaging.**
   - Use Kafka CLI to produce/consume on the topics that Spring Cloud Bus uses, verifying message flows.

3. **Scale services.**
   - Run multiple instances of `UserService`. All instances automatically get updated configs after a single bus refresh event.

---

## **Conclusion**
In this lab, you:
- **Set up a Spring Cloud Config Server** (with Bus + Kafka) on **port 8888**.
- **Configured** one or more microservices (`UserService`, `ProductService`) to fetch config from the server and **auto-refresh** changes.
- **Leveraged Spring Cloud Bus** (via **Kafka**) to broadcast configuration updates, ensuring real-time changes with minimal overhead.

This architecture significantly **simplifies config management** across distributed services, enabling near-instant synchronization of property changes without restarts.

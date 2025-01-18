# **Lab 9: Set Up Event-Based Messaging Between Microservices Using Spring Cloud Bus**

## **Objective**
Learn how to use Spring Cloud Bus to enable event-based messaging between microservices. Set up a configuration change propagation system to notify multiple services dynamically.

---

## **Lab Steps**

### **Part 1: Setting Up the Configuration Server**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-server`
     - **Name**: `ConfigServer`
     - **Dependencies**:
       - Spring Cloud Config Server
       - Spring Boot Actuator
       - Spring Cloud Bus
       - Spring Cloud Stream Kafka
   - Click **Generate** to download the project zip file.
   - Extract the zip file into a folder named `ConfigServer`.

2. **Import the project into your IDE.**

3. **Enable Spring Cloud Config Server.**
   - Open the `ConfigServerApplication.java` file and add the `@EnableConfigServer` annotation:
     ```java
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
   - Create `application.properties` in `src/main/resources` and add:
     ```properties
     server.port=8888
     spring.application.name=config-server
     spring.cloud.config.server.git.uri=https://github.com/<your-username>/config-repo
     spring.cloud.bus.enabled=true
     spring.cloud.stream.kafka.binder.brokers=localhost:9092
     ```

5. **Run the Config Server.**
   - Start the application:
     ```bash
     ./mvnw spring-boot:run
     ```

---

### **Part 2: Setting Up the Kafka Broker**

6. **Download and install Apache Kafka.**
   - Visit [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads) and download the latest version.
   - Extract the downloaded file to a folder (e.g., `/opt/kafka`).

7. **Start Zookeeper.**
   - Navigate to the Kafka folder and run:
     ```bash
     ./bin/zookeeper-server-start.sh config/zookeeper.properties
     ```

8. **Start Kafka.**
   - In another terminal, run:
     ```bash
     ./bin/kafka-server-start.sh config/server.properties
     ```

9. **Verify Kafka is running.**
   - List Kafka topics:
     ```bash
     ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
     ```

---

### **Part 3: Setting Up a Client Service**

10. **Generate a new Spring Boot project for `UserService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Artifact Id**: `user-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Config Client
        - Spring Cloud Bus
        - Spring Cloud Stream Kafka
    - Extract the downloaded zip file into a folder named `UserService`.

11. **Import the `UserService` project into your IDE.**

12. **Configure the `UserService` to use the Config Server.**
    - Create `application.properties` in `src/main/resources` and add:
      ```properties
      server.port=8081
      spring.application.name=user-service
      spring.cloud.config.uri=http://localhost:8888
      spring.cloud.bus.enabled=true
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      ```

13. **Create a REST controller to display configuration values.**
    - Create `ConfigController.java` in `src/main/java/com/microservices/userservice`:
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

14. **Run the `UserService` application.**
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

15. **Verify configuration retrieval.**
    - Access:
      ```bash
      http://localhost:8081/message
      ```
    - Confirm the `message` property from the Config Server is displayed.

---

### **Part 4: Testing Event-Based Messaging**

16. **Update the configuration file in the Git repository.**
    - Modify the `message` property in the configuration:
      ```yaml
      message: "Updated message from Config Server"
      ```
    - Commit and push the changes:
      ```bash
      git add .
      git commit -m "Updated message property"
      git push origin main
      ```

17. **Trigger a refresh event.**
    - Use a POST request to trigger the refresh:
      ```bash
      curl -X POST http://localhost:8888/actuator/bus-refresh
      ```

18. **Verify the updated configuration in `UserService`.**
    - Access:
      ```bash
      http://localhost:8081/message
      ```
    - Confirm the updated `message` property is displayed.

---

### **Part 5: Adding Another Client Service**

19. **Generate a new Spring Boot project for `ProductService`.**
    - Repeat steps 10–15 with the following differences:
      - **Artifact Id**: `product-service`
      - **Server port**: `8082`.

20. **Verify configuration sync between services.**
    - Trigger another refresh event using `/actuator/bus-refresh`.
    - Confirm both `UserService` and `ProductService` reflect the updated configuration.

---

### **Optional Exercises**

1. **Add a new property to the configuration.**
   - Add a new property (e.g., `service.version`) in the Git repository.
   - Verify that the new property is propagated dynamically to all services.

2. **Test Kafka messaging manually.**
   - Use Kafka CLI to produce and consume messages from Spring Cloud Bus topics.

3. **Simulate service scaling.**
   - Deploy multiple instances of `UserService` and test configuration propagation.

---

# **Lab 10: Use Spring Cloud Stream to Implement Event-Driven Communication Between Microservices**

## **Objective**
Learn how to use Spring Cloud Stream to implement event-driven communication between microservices. Configure Kafka as the message broker, publish events from one service, and consume them in another service.

---

## **Lab Steps**

### **Part 1: Installing and Running Kafka on Windows**

1. **Install Java (required for Kafka).**
   - **Verify Java installation:**
     - Open a Command Prompt and run:
       ```cmd
       java -version
       ```
     - If Java is not installed, download and install Java JDK 17 from [AdoptOpenJDK](https://adoptopenjdk.net/).
   - Add Java to the system `Path` if necessary:
     1. Go to **Environment Variables**.
     2. Add the path to the JDK’s `bin` folder (e.g., `C:\Program Files\Java\jdk-17\bin`).

2. **Download Kafka for Windows.**
   - Visit [Kafka Downloads](https://kafka.apache.org/downloads) and download the latest stable version (e.g., `kafka_2.13-3.4.0.tgz`).

3. **Extract Kafka.**
   - Use an archive extraction tool (e.g., WinRAR, 7-Zip) to extract Kafka to a folder, such as `C:\Kafka`.

4. **Set up Kafka configuration.**
   - Open the `config\server.properties` file in a text editor and update the following:
     ```properties
     log.dirs=C:/Kafka/kafka-logs
     ```

5. **Start Zookeeper.**
   - Open a Command Prompt, navigate to the Kafka folder, and run:
     ```cmd
     bin\windows\zookeeper-server-start.bat config\zookeeper.properties
     ```
   - Keep this terminal open.

6. **Start the Kafka broker.**
   - Open another Command Prompt, navigate to the Kafka folder, and run:
     ```cmd
     bin\windows\kafka-server-start.bat config\server.properties
     ```

7. **Verify that Kafka is running.**
   - Use the following command to list the current Kafka topics:
     ```cmd
     bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
     ```
   - If no topics are listed, Kafka is running but has no topics yet.

8. **Create a Kafka topic.**
   - Run the following command to create a topic named `order-events`:
     ```cmd
     bin\windows\kafka-topics.bat --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
     ```

9. **Verify the topic creation.**
    - List all Kafka topics:
      ```cmd
      bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
      ```
    - Ensure the `order-events` topic is listed.

---

### **Part 2: Setting Up the Producer Microservice**

10. **Generate a new Spring Boot project for `OrderService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Group Id**: `com.microservices`
      - **Artifact Id**: `order-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the downloaded zip file into a folder named `OrderService`.

11. **Import the project into your IDE.**
    - Open your IDE and import the `OrderService` project as a Maven project.

12. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
    - Add the following properties:
      ```properties
      spring.application.name=order-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.output.destination=order-events
      ```

13. **Create an event model.**
    - Create a new class `OrderEvent.java`:
      ```java
      package com.microservices.orderservice;

      public class OrderEvent {
          private String orderId;
          private String status;

          public OrderEvent(String orderId, String status) {
              this.orderId = orderId;
              this.status = status;
          }

          // Getters and setters
          public String getOrderId() { return orderId; }
          public void setOrderId(String orderId) { this.orderId = orderId; }
          public String getStatus() { return status; }
          public void setStatus(String status) { this.status = status; }
      }
      ```

14. **Create a message producer.**
    - Create a new file `OrderProducer.java`:
      ```java
      package com.microservices.orderservice;

      import org.springframework.cloud.stream.function.StreamBridge;
      import org.springframework.stereotype.Component;

      @Component
      public class OrderProducer {

          private final StreamBridge streamBridge;

          public OrderProducer(StreamBridge streamBridge) {
              this.streamBridge = streamBridge;
          }

          public void sendOrderEvent(OrderEvent event) {
              streamBridge.send("output", event);
          }
      }
      ```

15. **Add a REST controller to trigger events.**
    - Create a new file `OrderController.java`:
      ```java
      package com.microservices.orderservice;

      import org.springframework.web.bind.annotation.PostMapping;
      import org.springframework.web.bind.annotation.RequestBody;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class OrderController {

          private final OrderProducer orderProducer;

          public OrderController(OrderProducer orderProducer) {
              this.orderProducer = orderProducer;
          }

          @PostMapping("/orders")
          public String createOrder(@RequestBody OrderEvent orderEvent) {
              orderProducer.sendOrderEvent(orderEvent);
              return "Order event sent for order ID: " + orderEvent.getOrderId();
          }
      }
      ```

16. **Run the `OrderService` application.**
    - Start the application using:
      ```cmd
      mvnw spring-boot:run
      ```

17. **Test the event publishing.**
    - Use Postman to send a POST request to:
      ```
      http://localhost:8080/orders
      ```
      - Request body:
        ```json
        {
          "orderId": "123",
          "status": "CREATED"
        }
        ```

---

### **Part 3: Setting Up the Consumer Microservice**

18. **Generate a new Spring Boot project for `NotificationService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Artifact Id**: `notification-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the zip file into a folder named `NotificationService`.

19. **Import the project into your IDE.**
    - Open your IDE and import the `NotificationService` project as a Maven project.

20. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
    - Add the following properties:
      ```properties
      spring.application.name=notification-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.input.destination=order-events
      ```

21. **Create a message consumer.**
    - Create a new file `OrderEventConsumer.java`:
      ```java
      package com.microservices.notificationservice;

      import org.springframework.context.annotation.Bean;
      import org.springframework.stereotype.Component;
      import java.util.function.Consumer;

      @Component
      public class OrderEventConsumer {

          @Bean
          public Consumer<OrderEvent> input() {
              return orderEvent -> {
                  System.out.println("Received order event: " + orderEvent);
              };
          }
      }
      ```

22. **Create an event model for the consumer.**
    - Create a class `OrderEvent.java` (similar to the producer’s `OrderEvent`).

23. **Run the `NotificationService` application.**
    - Start the application using:
      ```cmd
      mvnw spring-boot:run
      ```

24. **Test the event-driven communication.**
    - Send a POST request to `OrderService` and verify that `NotificationService` logs the event.

---

### **Optional Exercises**

1. Add a dead-letter topic for failed messages.
2. Extend the `OrderEvent` model with additional fields (e.g., `timestamp`) and test end-to-end.

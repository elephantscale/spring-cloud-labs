# **Lab 10: Use Spring Cloud Stream to Implement Event-Driven Communication Between Microservices**

## **Objective**
Learn how to use Spring Cloud Stream to implement event-driven communication between microservices. Configure Kafka as the message broker, publish events from one service, and consume them in another service.

---

## **Lab Steps**

### **Part 1: Installing and Running Kafka**

1. **Install Java (required for Kafka).**
   - Verify Java installation:
     ```cmd
     java -version
     ```
     - If Java is not installed, download and install JDK 17 from [AdoptOpenJDK](https://adoptopenjdk.net/).
   - Add Java to the system `Path` if necessary:
     - Go to **Environment Variables** and add the JDK `bin` folder to the `Path`.

2. **Download Kafka.**
   - Visit [Kafka Downloads](https://kafka.apache.org/downloads) and download the latest stable version (e.g., `kafka_2.13-3.4.0.tgz`).

3. **Extract Kafka.**
   - Extract the Kafka archive to a folder, such as `/opt/kafka`.

4. **Start Zookeeper.**
   - Navigate to the Kafka folder and run:
     ```bash
     ./bin/zookeeper-server-start.sh config/zookeeper.properties
     ```

5. **Start the Kafka broker.**
   - Open another terminal and run:
     ```bash
     ./bin/kafka-server-start.sh config/server.properties
     ```

6. **Create a Kafka topic.**
   - Run the following command to create a topic named `order-events`:
     ```bash
     ./bin/kafka-topics.sh --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
     ```

7. **Verify the topic creation.**
   - List all Kafka topics:
     ```bash
     ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
     ```
   - Ensure the `order-events` topic is listed.

---

### **Part 2: Setting Up the Producer Microservice**

8. **Generate a new Spring Boot project for `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Stream
       - Spring Cloud Stream Kafka Binder
   - Extract the downloaded zip file into a folder named `OrderService`.

9. **Import the project into your IDE.**

10. **Configure Spring Cloud Stream for Kafka in `application.yml`.**
    - Create `application.yml` in `src/main/resources` and add:
      ```yaml
      spring:
        application:
          name: order-service
        cloud:
          stream:
            kafka:
              binder:
                brokers: localhost:9092
            bindings:
              output:
                destination: order-events
      ```

11. **Create an event model.**
    - Add `OrderEvent.java`:
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

12. **Create a message producer.**
    - Add `OrderProducer.java`:
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

13. **Add a REST controller to trigger events.**
    - Add `OrderController.java`:
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

14. **Run the `OrderService` application.**
    ```bash
    ./mvnw spring-boot:run
    ```

15. **Test the event publishing.**
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

16. **Generate a new Spring Boot project for `NotificationService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Artifact Id**: `notification-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the zip file into a folder named `NotificationService`.

17. **Import the project into your IDE.**

18. **Configure Spring Cloud Stream for Kafka in `application.yml`.**
    - Create `application.yml` in `src/main/resources` and add:
      ```yaml
      spring:
        application:
          name: notification-service
        cloud:
          stream:
            kafka:
              binder:
                brokers: localhost:9092
            bindings:
              input:
                destination: order-events
      ```

19. **Create a message consumer.**
    - Add `OrderEventConsumer.java`:
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

20. **Create an event model.**
    - Add `OrderEvent.java` (same as the producer's model).

21. **Run the `NotificationService` application.**
    ```bash
    ./mvnw spring-boot:run
    ```

22. **Test the event-driven communication.**
    - Send a POST request to `OrderService` and verify that `NotificationService` logs the event.

---

### **Optional Exercises**

1. **Add a dead-letter topic.**
   - Configure a dead-letter queue for failed messages.

2. **Enhance the event model.**
   - Extend `OrderEvent` with additional fields (e.g., `timestamp`) and test.

3. **Simulate scaling.**
   - Deploy multiple instances of `NotificationService` and observe message distribution.

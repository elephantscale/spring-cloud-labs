# **Lab 10: Use Spring Cloud Stream to Implement Event-Driven Communication Between Microservices**

## **Objective**
Learn how to use Spring Cloud Stream to implement event-driven communication between microservices. Configure Kafka as the message broker, publish events from one service, and consume them in another service.

---

## **Lab Steps**

### **Part 1: Installing and Running Kafka on Linux**

1. **Update Linux packages.**
   - Open a terminal and run:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

2. **Install Java (required for Kafka).**
   - Check if Java is already installed:
     ```bash
     java -version
     ```
   - If Java is not installed, run:
     ```bash
     sudo apt install openjdk-17-jdk -y
     ```
   - Verify the installation:
     ```bash
     java -version
     ```

3. **Download Kafka.**
   - Visit [Kafka Downloads](https://kafka.apache.org/downloads) and copy the link for the latest stable version.
   - Use `wget` to download Kafka:
     ```bash
     wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz
     ```

4. **Extract the Kafka archive.**
   - Run the following command to extract Kafka:
     ```bash
     tar -xvzf kafka_2.13-3.4.0.tgz
     ```
   - Move the extracted folder to `/opt`:
     ```bash
     sudo mv kafka_2.13-3.4.0 /opt/kafka
     ```

5. **Set up environment variables.**
   - Open the `.bashrc` file:
     ```bash
     nano ~/.bashrc
     ```
   - Add the following lines at the end:
     ```bash
     export KAFKA_HOME=/opt/kafka
     export PATH=$PATH:$KAFKA_HOME/bin
     ```
   - Save and apply the changes:
     ```bash
     source ~/.bashrc
     ```

6. **Start Zookeeper.**
   - Kafka requires Zookeeper to manage cluster information.
   - Run the following command to start Zookeeper:
     ```bash
     zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
     ```
   - Keep this terminal open or run it in a new terminal with `screen` or `tmux`.

7. **Start the Kafka broker.**
   - Open another terminal and start Kafka:
     ```bash
     kafka-server-start.sh /opt/kafka/config/server.properties
     ```

8. **Verify that Kafka is running.**
   - Use the following command to list the current Kafka topics:
     ```bash
     kafka-topics.sh --list --bootstrap-server localhost:9092
     ```
   - If no topics are listed, Kafka is running but has no topics yet.

9. **Create a Kafka topic.**
   - Run the following command to create a topic named `order-events`:
     ```bash
     kafka-topics.sh --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
     ```

10. **Verify the topic creation.**
    - List all Kafka topics:
      ```bash
      kafka-topics.sh --list --bootstrap-server localhost:9092
      ```
    - Ensure the `order-events` topic is listed.

---

### **Part 2: Setting Up the Producer Microservice**

11. **Generate a new Spring Boot project for `OrderService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Group Id**: `com.microservices`
      - **Artifact Id**: `order-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the downloaded zip file into a folder named `OrderService`.

12. **Import the project into your IDE.**
    - Open your IDE and import the `OrderService` project as a Maven project.

13. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
    - Add the following properties:
      ```properties
      spring.application.name=order-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.output.destination=order-events
      ```

14. **Create an event model.**
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

15. **Create a message producer.**
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

16. **Add a REST controller to trigger events.**
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

17. **Run the `OrderService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

18. **Test the event publishing.**
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

19. **Generate a new Spring Boot project for `NotificationService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Artifact Id**: `notification-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the zip file into a folder named `NotificationService`.

20. **Import the project into your IDE.**
    - Open your IDE and import the `NotificationService` project as a Maven project.

21. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
    - Add the following properties:
      ```properties
      spring.application.name=notification-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.input.destination=order-events
      ```

22. **Create a message consumer.**
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

23. **Create an event model for the consumer.**
    - Create a class `OrderEvent.java` (similar to the producer’s `OrderEvent`).

24. **Run the `NotificationService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

25. **Test the event-driven communication.**
    - Send a POST request to `OrderService` and verify that `NotificationService` logs the event.

---

### **Optional Exercises**

1. Add a dead-letter topic for failed messages.
2. Extend the `OrderEvent` model with additional fields (e.g., `timestamp`) and test end-to-end.


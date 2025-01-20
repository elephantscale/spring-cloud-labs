# **Lab 10: Use Spring Cloud Stream to Implement Event-Driven Communication Between Microservices (Spring Boot 3.4.1)**

## **Objective**
Learn how to use **Spring Cloud Stream** with **Kafka** as the message broker to enable event-driven communication among microservices. You’ll create a producer (publishing `OrderEvent`s) and a consumer (receiving those events) in Spring Boot **3.4.1**.

---

## **Lab Steps**

### **Part 1: Installing and Running Kafka**

1. **Install Java (required for Kafka).**
   - Verify Java by running:
     ```bash
     java -version
     ```
   - If missing, install **JDK 17** (e.g., from [AdoptOpenJDK](https://adoptium.net/)).

2. **Download Kafka.**
   - Go to [Kafka Downloads](https://kafka.apache.org/downloads) and get the latest stable release (e.g., `kafka_2.13-3.4.0.tgz`).

3. **Extract Kafka.**
   - For example:
     ```bash
     tar -xvzf kafka_2.13-3.4.0.tgz -C /opt/kafka
     ```

4. **Start Zookeeper.**
   - In one terminal:
     ```bash
     cd /opt/kafka
     ./bin/zookeeper-server-start.sh config/zookeeper.properties
     ```

5. **Start the Kafka broker.**
   - In another terminal:
     ```bash
     cd /opt/kafka
     ./bin/kafka-server-start.sh config/server.properties
     ```

6. **Create a Kafka topic.**
   - For example, to create `order-events`:
     ```bash
     ./bin/kafka-topics.sh --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
     ```

7. **Verify the topic creation.**
   - List Kafka topics:
     ```bash
     ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
     ```
   - Confirm `order-events` is present.

---

### **Part 2: Setting Up the Producer Microservice (`OrderService`)**

8. **Generate a new Spring Boot project for `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - **Spring Boot Version**: **3.4.1**
   - **Group Id**: `com.microservices`
   - **Artifact Id**: `order-service`
   - **Dependencies**:
     - Spring Web
     - Spring Cloud Stream
     - Spring Cloud Stream Kafka Binder
   - Extract to `OrderService`.

9. **Import `OrderService` into your IDE.**

10. **Configure Spring Cloud Stream for Kafka.**
    - In `src/main/resources/application.properties`:
      ```properties
      spring.application.name=order-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.output.destination=order-events
      ```
    - This means the **binding** name is `output`, publishing to the topic `order-events`.

11. **Create an event model.**
    - In `OrderEvent.java`:
      ```java
      package com.microservices.orderservice;

      public class OrderEvent {
          private String orderId;
          private String status;

          public OrderEvent(String orderId, String status) {
              this.orderId = orderId;
              this.status = status;
          }

          public String getOrderId() { return orderId; }
          public void setOrderId(String orderId) { this.orderId = orderId; }

          public String getStatus() { return status; }
          public void setStatus(String status) { this.status = status; }

          @Override
          public String toString() {
              return "OrderEvent{" +
                     "orderId='" + orderId + '\'' +
                     ", status='" + status + '\'' +
                     '}';
          }
      }
      ```

12. **Create a message producer.**
    - In `OrderProducer.java`:
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
              // "output" matches the binding name in application.properties
              streamBridge.send("output", event);
          }
      }
      ```

13. **Add a REST controller to trigger events.**
    - In `OrderController.java`:
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

14. **Run `OrderService`.**
    - From `OrderService`:
      ```bash
      ./mvnw spring-boot:run
      ```
    - It should start on port **8080** (by default).

15. **Test the event publishing.**
    - Use Postman or curl:
      ```bash
      curl -X POST \
           -H "Content-Type: application/json" \
           -d '{"orderId":"123","status":"CREATED"}' \
           http://localhost:8080/orders
      ```
    - Check the console logs for any errors and confirm the event is produced to Kafka.

---

### **Part 3: Setting Up the Consumer Microservice (`NotificationService`)**

16. **Generate a new Spring Boot project for `NotificationService`.**
    - **Artifact Id**: `notification-service`
    - **Dependencies**:
      - Spring Web
      - Spring Cloud Stream
      - Spring Cloud Stream Kafka Binder
   - Extract to `NotificationService`.

17. **Import `NotificationService` into your IDE.**

18. **Configure Spring Cloud Stream for Kafka.**
    - In `application.properties`:
      ```properties
      spring.application.name=notification-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.input.destination=order-events
      ```
    - This indicates the **binding** `input` is listening to the same `order-events` topic.

19. **Create a message consumer.**
    - In `OrderEventConsumer.java`:
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
    - In `OrderEvent.java` (same as in `OrderService`):
      ```java
      package com.microservices.notificationservice;

      public class OrderEvent {
          private String orderId;
          private String status;

          // constructors, getters, setters, toString
      }
      ```

21. **Run `NotificationService`.**
    - From `NotificationService` folder:
      ```bash
      ./mvnw spring-boot:run
      ```
    - It also starts on **8080** by default (or define another port in application.properties if you wish).

22. **Test the event-driven communication.**
    - Post an order event to `OrderService`:
      ```bash
      curl -X POST \
           -H "Content-Type: application/json" \
           -d '{"orderId":"456","status":"CREATED"}' \
           http://localhost:8080/orders
      ```
    - In the `NotificationService` logs, you should see something like:
      ```
      Received order event: OrderEvent{orderId='456', status='CREATED'}
      ```

---

## **Optional Exercises**

1. **Add a dead-letter topic.**
   - Configure a DLT for messages that cannot be processed.

2. **Enhance the event model.**
   - Add fields (e.g., `timestamp`, `customerId`) to `OrderEvent` in both producer and consumer to handle more complex data.

3. **Simulate scaling.**
   - Run multiple `NotificationService` instances to observe how Kafka distributes messages.

---

## **Conclusion**
By completing this lab, you have:
- **Installed and run** Kafka locally (Zookeeper + Kafka broker).
- **Built a producer** (`OrderService`) that publishes messages to a topic (`order-events`).
- **Created a consumer** (`NotificationService`) that listens on the same topic.
- **Verified** event-driven communication works seamlessly with **Spring Cloud Stream**.

This lays the foundation for decoupled microservices, letting them react to events asynchronously and scale independently.

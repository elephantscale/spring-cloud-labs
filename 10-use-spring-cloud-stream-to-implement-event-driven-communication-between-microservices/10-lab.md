# **Lab 10: Use Spring Cloud Stream to Implement Event-Driven Communication Between Microservices**

## **Objective**
Learn how to use Spring Cloud Stream to implement event-driven communication between microservices. Configure Kafka as the message broker, publish events from one service, and consume them in another service.

---

## **Lab Steps**

### **Part 1: Setting Up Kafka**

1. **Download and install Apache Kafka.**
   - Visit [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads) and download the latest version.
   - Extract the downloaded file to a directory of your choice.

2. **Start Zookeeper.**
   - Open a terminal, navigate to the Kafka `bin` directory, and run:
     ```bash
     bin/zookeeper-server-start.sh config/zookeeper.properties
     ```

3. **Start Kafka.**
   - Open another terminal and run:
     ```bash
     bin/kafka-server-start.sh config/server.properties
     ```

4. **Create a Kafka topic.**
   - Use the Kafka CLI to create a topic named `order-events`:
     ```bash
     bin/kafka-topics.sh --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
     ```

5. **Verify the topic creation.**
   - List all topics to confirm:
     ```bash
     bin/kafka-topics.sh --list --bootstrap-server localhost:9092
     ```

---

### **Part 2: Setting Up the Producer Microservice**

6. **Generate a new Spring Boot project for the `OrderService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `order-service`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Stream
       - Spring Cloud Stream Kafka Binder
   - Extract the downloaded zip file into a folder named `OrderService`.

7. **Import the project into your IDE.**
   - Open your IDE and import the `OrderService` project as a Maven project.

8. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
   - Add the following properties:
     ```properties
     spring.application.name=order-service
     spring.cloud.stream.kafka.binder.brokers=localhost:9092
     spring.cloud.stream.bindings.output.destination=order-events
     ```

9. **Create an event model.**
   - Create a new class `OrderEvent.java` in the `src/main/java/com/microservices/orderservice` folder:
     ```java
     package com.microservices.orderservice;

     public class OrderEvent {
         private String orderId;
         private String status;

         public OrderEvent(String orderId, String status) {
             this.orderId = orderId;
             this.status = status;
         }

         public String getOrderId() {
             return orderId;
         }

         public void setOrderId(String orderId) {
             this.orderId = orderId;
         }

         public String getStatus() {
             return status;
         }

         public void setStatus(String status) {
             this.status = status;
         }
     }
     ```

10. **Create a message producer.**
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

11. **Add a REST controller to trigger events.**
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

12. **Run the `OrderService` application.**
    - Start the application using:
      ```bash
      ./mvnw spring-boot:run
      ```

13. **Test the event publishing.**
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

14. **Generate a new Spring Boot project for `NotificationService`.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Artifact Id**: `notification-service`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Stream
        - Spring Cloud Stream Kafka Binder
    - Extract the downloaded zip file into a folder named `NotificationService`.

15. **Import the project into your IDE.**
    - Open your IDE and import the `NotificationService` project as a Maven project.

16. **Configure Spring Cloud Stream for Kafka in `application.properties`.**
    - Add the following properties:
      ```properties
      spring.application.name=notification-service
      spring.cloud.stream.kafka.binder.brokers=localhost:9092
      spring.cloud.stream.bindings.input.destination=order-events
      ```

17. **Create a message consumer.**
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

18. **Create an event model for the consumer.**
    - Create a class `OrderEvent.java` (similar to the producer’s `OrderEvent`).

19. **Run the `NotificationService` application.**
    - Start the `NotificationService` using:
      ```bash
      ./mvnw spring-boot:run
      ```

---

### **Part 4: Testing the Event-Driven Communication**

20. **Test the event flow.**
    - Send a POST request to `OrderService` as described in step 13.
    - Verify that the `NotificationService` logs the event details in the console.

---

### **Optional Exercises (20 mins)**

1. **Add additional fields to the `OrderEvent` model.**
   - Extend the event with additional fields (e.g., `orderAmount`) and test the flow.

2. **Integrate error handling in the consumer.**
   - Implement a dead-letter topic to handle failed message processing.


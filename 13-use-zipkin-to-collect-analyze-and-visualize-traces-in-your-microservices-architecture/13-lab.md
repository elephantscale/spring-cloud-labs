# **Lab 13: Use Zipkin to Collect, Analyze, and Visualize Traces in Your Microservices Architecture**

## **Objective**
Learn how to integrate Spring Cloud Sleuth with Zipkin to collect and visualize distributed traces in a microservices architecture. Use the Zipkin dashboard to analyze request flow and latency across services.

---

## **Lab Steps**

### **Part 1: Installing and Running Zipkin**

1. **Ensure Java is installed (required for running Zipkin).**
   - Check if Java is installed by running:
     ```bash
     java -version
     ```
   - If not installed:
     - Download and install JDK 17 from [AdoptOpenJDK](https://adoptium.net/).
     - Verify the installation:
       ```bash
       java -version
       ```

2. **Download the Zipkin Server.**
   - Visit [Zipkin Quickstart](https://zipkin.io/pages/quickstart) and download the latest Zipkin JAR file.
   - Save the file (e.g., `zipkin-server-2.23.16-exec.jar`) to a directory like `~/zipkin` or `C:\Zipkin`.

3. **Run the Zipkin server.**
   - Open a terminal or command prompt, navigate to the directory containing the JAR file, and start Zipkin:
     ```bash
     java -jar zipkin-server-2.23.16-exec.jar
     ```
   - Zipkin will start on port `9411`.

4. **Verify that Zipkin is running.**
   - Open a browser and navigate to:
     ```
     http://localhost:9411
     ```
   - Confirm that the Zipkin dashboard is displayed.

---

### **Part 2: Configuring the Producer Microservice**

5. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1**.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Boot Actuator
       - Spring Cloud Sleuth
   - Extract the zip file into a folder named `UserService`.

6. **Import the `UserService` project into your IDE.**

7. **Configure `UserService` to send traces to Zipkin.**
   - Add the following properties in `src/main/resources/application.yml`:
     ```yaml
     server:
       port: 8081
     spring:
       application:
         name: user-service
       zipkin:
         base-url: http://localhost:9411
       sleuth:
         sampler:
           probability: 1.0
     ```

8. **Create a REST controller in `UserService`.**
   - Add the file `UserController.java`:
     ```java
     package com.microservices.userservice;

     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class UserController {

         @GetMapping("/users")
         public String getUsers() {
             return "List of users from UserService";
         }
     }
     ```

9. **Run the `UserService`.**
   - Start the application using:
     ```bash
     ./mvnw spring-boot:run
     ```

10. **Test the `/users` endpoint.**
    - Access `http://localhost:8081/users` and verify the response.

---

### **Part 3: Configuring the Consumer Microservice**

11. **Generate a new Spring Boot project for `OrderService`.**
    - Configure the project:
      - **Artifact Id**: `order-service`
      - **Dependencies**:
        - Spring Web
        - Spring Boot Actuator
        - Spring Cloud Sleuth
        - Spring WebClient
    - Extract the zip file into a folder named `OrderService`.

12. **Import the `OrderService` project into your IDE.**

13. **Configure `OrderService` to send traces to Zipkin.**
    - Add the following properties in `src/main/resources/application.yml`:
      ```yaml
      server:
        port: 8082
      spring:
        application:
          name: order-service
        zipkin:
          base-url: http://localhost:9411
        sleuth:
          sampler:
            probability: 1.0
      ```

14. **Create a REST controller in `OrderService`.**
    - Add the file `OrderController.java`:
      ```java
      package com.microservices.orderservice;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;
      import org.springframework.web.reactive.function.client.WebClient;

      @RestController
      public class OrderController {

          @Autowired
          private WebClient.Builder webClientBuilder;

          @GetMapping("/orders")
          public String getOrders() {
              String users = webClientBuilder.build()
                      .get()
                      .uri("http://localhost:8081/users")
                      .retrieve()
                      .bodyToMono(String.class)
                      .block();

              return "Orders from OrderService and Users: " + users;
          }
      }
      ```

15. **Add WebClient configuration to `OrderService`.**
    - Add the following bean in `OrderServiceApplication.java`:
      ```java
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.context.annotation.Bean;
      import org.springframework.web.reactive.function.client.WebClient;

      @SpringBootApplication
      public class OrderServiceApplication {

          public static void main(String[] args) {
              SpringApplication.run(OrderServiceApplication.class, args);
          }

          @Bean
          public WebClient.Builder webClientBuilder() {
              return WebClient.builder();
          }
      }
      ```

16. **Run the `OrderService`.**
    ```bash
    ./mvnw spring-boot:run
    ```

17. **Test the `/orders` endpoint.**
    - Access `http://localhost:8082/orders` and verify that it fetches data from `UserService`.

---

### **Part 4: Visualizing Traces in Zipkin**

18. **Generate traffic between the services.**
    - Use Postman or a browser to call the `/orders` endpoint multiple times.

19. **Access the Zipkin dashboard.**
    - Open a browser and navigate to:
      ```
      http://localhost:9411
      ```

20. **Search for traces.**
    - Click on **Find Traces** in the Zipkin dashboard.
    - View the traces associated with `order-service` and `user-service`.

21. **Analyze a specific trace.**
    - Click on a trace to view its details, including spans, timing, and service dependencies.

---

### **Optional Exercises**

1. **Add a third microservice.**
   - Create a `ProductService` and integrate it with the existing services.
   - Visualize the end-to-end trace in Zipkin.

2. **Simulate a service failure.**
   - Stop `UserService` and observe how Zipkin visualizes the error in the trace.

---

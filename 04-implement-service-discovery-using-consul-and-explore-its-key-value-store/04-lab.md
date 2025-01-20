# **Lab 4: Implement Service Discovery Using Consul and Explore Its Key-Value Store (Spring Boot 3.4.1)**

## **Objective**
Learn how to use **HashiCorp Consul** to enable dynamic service discovery and leverage its KV (Key-Value) store for storing and retrieving configuration data. You will register **Spring Boot 3.4.1** microservices with Consul and explore how they can fetch dynamic properties from Consul’s KV store.

---

## **Lab Steps**

### **Part 1: Installing and Running Consul**

1. **Download the Consul binary.**
   - Visit [Consul Downloads](https://developer.hashicorp.com/consul/downloads) and download the latest binary for your operating system.

2. **Extract the Consul binary.**
   - **Windows**:
     - Unzip the file (e.g., `consul_<version>_windows_amd64.zip`) to a folder like `C:\HashiCorp\Consul`.
   - **macOS/Linux**:
     - Extract it to `/usr/local/bin` or another suitable directory:
       ```bash
       unzip consul_<version>_<platform>.zip -d /usr/local/bin/
       ```

3. **Add Consul to the PATH.**
   - **Windows**:
     - Edit **Path** in Environment Variables to include the folder with `consul.exe`.
   - **macOS/Linux**:
     - In your shell config:
       ```bash
       export PATH=$PATH:/usr/local/bin/consul
       ```
     - Reload:
       ```bash
       source ~/.bashrc
       ```

4. **Start Consul in development mode.**
   - In a terminal:
     ```bash
     consul agent -dev
     ```
   - Consul listens on port **8500** by default and runs in memory.

5. **Access the Consul UI.**
   - Go to:
     ```
     http://127.0.0.1:8500
     ```
   - You should see the Consul dashboard (no registered services yet).

6. **Add a key-value pair to the KV store.**
   - From the **Key/Value** tab in the UI:
     - **Key**: `config/sample-service`
     - **Value**:
       ```json
       {
         "greeting": "Hello from Consul KV Store!"
       }
       ```
   - Save the changes.

7. **Verify KV store retrieval using CLI.**
   - In a terminal:
     ```bash
     consul kv get config/sample-service
     ```
   - Confirm it matches the JSON value set in the UI.

---

### **Part 2: Setting Up a Consul-Enabled Microservice**

8. **Generate a new Spring Boot project for `ConsulServer`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `consul-server`
     - **Name**: `ConsulServer`
     - **Package Name**: `com.microservices.consulserver`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Consul Discovery
       - Spring Boot Actuator
   - Extract into a folder named `ConsulServer`.

9. **Import the `ConsulServer` project into your IDE.**

10. **Enable Consul Discovery.**
    - In `ConsulServerApplication.java`:
      ```java
      package com.microservices.consulserver;

      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

      @SpringBootApplication
      @EnableDiscoveryClient
      public class ConsulServerApplication {
          public static void main(String[] args) {
              SpringApplication.run(ConsulServerApplication.class, args);
          }
      }
      ```

11. **Configure Consul in `application.properties`.**
    - Create `src/main/resources/application.properties`:
      ```properties
      spring.application.name=consul-server
      spring.cloud.consul.host=127.0.0.1
      spring.cloud.consul.port=8500

      server.port=8081
      ```

12. **Add a sample REST endpoint.**
    - In `src/main/java/com/microservices/consulserver/ConsulController.java`:
      ```java
      package com.microservices.consulserver;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ConsulController {

          @GetMapping("/greeting")
          public String greeting() {
              return "Hello from Consul Server!";
          }
      }
      ```

13. **Run `ConsulServer`.**
    - From the `ConsulServer` folder:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check Consul UI at:
      ```
      http://127.0.0.1:8500
      ```
    - Under **Services**, you should see `consul-server`.

---

### **Part 3: Using the Consul KV Store in Your Microservice**

14. **Fetch KV store values dynamically.**
    - Modify `ConsulController` to retrieve a property from Consul:
      ```java
      package com.microservices.consulserver;

      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class ConsulController {

          @Value("${greeting:Default Greeting}")
          private String greeting;

          @GetMapping("/greeting")
          public String getGreeting() {
              return greeting;
          }
      }
      ```
    - Now, `greeting` will map to `config/sample-service`’s `greeting` if your application name is set to `consul-server` or you override the key.

15. **Test KV store retrieval.**
    - Access:
      ```
      http://localhost:8081/greeting
      ```
    - You should see **Hello from Consul KV Store!** if your key and property are correctly aligned.

---

### **Part 4: Adding Another Service**

16. **Generate another service, e.g. `UserService`.**
    - Repeat steps 8–11 with:
      - **Artifact Id**: `user-service`
      - **Name**: `UserService`
      - **Package Name**: `com.microservices.userservice`
      - Change `server.port=8082`.

17. **Verify both services in Consul.**
    - After running both:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check the Consul UI again. You should see `consul-server` and `user-service`.

---

## **Optional Exercises**

1. **Add environment-specific KV keys.**
   - E.g., `config/sample-service/dev` vs. `config/sample-service/prod`. Switch profiles to see different values.

2. **Simulate a service failure.**
   - Stop or kill one microservice and observe Consul automatically mark it as unhealthy or remove it.

3. **Use Consul for additional config.**
   - Store property sets for each environment or service in the KV store, retrieving them dynamically in your microservices.

---

## **Conclusion**
In this lab, you:
- **Installed and ran Consul** in development mode.
- **Created key-value pairs** for dynamic configuration.
- **Registered** one or more **Spring Boot 3.4.1** microservices with **Consul** using Spring Cloud Consul.
- **Fetched KV store data** to update microservices with dynamic configurations.

This pattern enables flexible service discovery, health checks, and a centralized configuration approach, paving the way for more advanced features (e.g., load balancing, auto-scaling, multi-datacenter replication). Enjoy building on Consul’s robust capabilities!

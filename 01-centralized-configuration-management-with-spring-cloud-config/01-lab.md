# **Lab 1: Centralized Configuration Management with Spring Cloud Config (Spring Boot 3.4.4)**

## **Objective**
Learn how to set up centralized configuration management for microservices using Spring Boot **3.4.4** and Spring Cloud Config. You will create:
1. A **Config Service** that reads a local property initially.
2. A **Config Server** that manages configurations from a Git repo.
3. A **Config Client** that fetches and refreshes properties dynamically from the server.

---

## **Lab Steps**

### **Part 1: Creating the Config Service (Client) as a Simple Microservice**

1. **Generate a new Spring Boot project for `ConfigService`.**
   - Open in a new tab (Right-click+R) [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: **3.4.4**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-service`
     - **Package Name**: `com.microservices.configservice`
     - **Dependencies**: 
       - Spring Web
   - Click **Generate** to download the project zip file.
   - Extract it into a folder named `ConfigService`.

2. **Import the project into your IDE.**
   - Open your IDE **(IntelliJ)** and import the `ConfigService` project as a Maven project.

3. **Add a local property in `application.properties`.**
   - In `src/main/resources/application.properties`, add:
     ```properties
     message=HelloWorld
     ```

4. **Create a controller to verify the property.**
   - In `src/main/java/com/microservices/configservice`, create `HelloController.java`:
     ```java
     package com.microservices.configservice;

     import org.springframework.beans.factory.annotation.Value;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class HelloController {

         @Value("${message:Default message}")
         private String message;

         @GetMapping("/message")
         public String getMessage() {
             return message;
         }
     }
     ```

5. **Run and test the Config Service.**
   - Use:
     ```bash
     ./mvnw spring-boot:run
     ```
   - Access `http://localhost:8080/message`. You should see **HelloWorld**.

---

### **Part 2: Creating a Git Repository for Centralized Config**

6. **Create a Git repository.**
   - Go to [GitHub](https://github.com/) (or another Git provider).
   - Create a repo named `config-repo` (public or private).

7. **Clone the repository locally.**
   - In a terminal:
     ```bash
     git clone https://github.com/<your-username>/config-repo.git
     cd config-repo
     ```

8. **Add configuration files to the repo.**
   - Create two files for now:
     - `application.properties`:
       ```properties
       message=Hello from Config Server!
       ```
     - `config-service.properties`:
       ```properties
       message=Hello from Config Service!
       ```
   - Commit and push:
     ```bash
     git add .
     git commit -m "Initial config files"
     git push origin main
     ```

---

### **Part 3: Setting Up the Spring Cloud Config Server**

9. **Generate a new Spring Boot project for `ConfigServer`.**
   - Open in a new tab (Right-click+R) [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: **3.4.4**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-server`
     - **Name**: `ConfigServer`
     - **Package Name**: `com.microservices.configserver`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Config Server
   - Click **Generate** and extract into `ConfigServer`.

10. **Import `ConfigServer` into your IDE.**

11. **Enable the Config Server.**
    - In `ConfigServerApplication.java`, add:
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

12. **Configure the Config Server to use your Git repo.**
    - In `src/main/resources/application.properties`, add:
      ```properties
      server.port=8888
      spring.application.name=config-server

      spring.cloud.config.server.git.uri=https://github.com/<your-username>/config-repo
      ```
    - Replace `<your-username>` with your actual GitHub username.

13. **Run and test the Config Server.**
    - Use:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check:
      - `http://localhost:8888/application/default`
      - `http://localhost:8888/config-service/default`
    - You should see your Git-based properties.

---

### **Part 4: Connecting `ConfigService` to the Config Server**

14. **Add Spring Cloud Config dependency to `ConfigService`.**
    - In `ConfigService`’s `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-config</artifactId>
      </dependency>
      ```

15. **Configure `ConfigService` to import configs from the server.**
    - In `application.properties` of `ConfigService`, add:
      ```properties
      spring.application.name=config-service
      spring.cloud.import=configserver:http://localhost:8888
      ```
    - This is the updated approach for Spring Boot 3.4.x / Spring Cloud 2023.0+.

16. **Restart the `ConfigService`.**
    - Again:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Check logs to ensure it pulls from `config-server`.

17. **Test dynamic retrieval.**
    - Access `http://localhost:8080/message`.
    - Now you should see `Hello from Config Service!` (based on `config-service.properties` in Git).

---

### **Part 5: Advanced Configuration Management**

18. **Add Spring Actuator to enable refresh.**
    - In `ConfigService`’s `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

19. **Expose the refresh endpoint.**
    - In `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

20. **Update a property in Git and refresh.**
    - Modify `config-service.properties` in your `config-repo`:
      ```properties
      message=Hello from Updated Config Server!
      ```
    - Commit and push:
      ```bash
      git add .
      git commit -m "Update message property"
      git push origin main
      ```

21. **Trigger a refresh and verify.**
    - Use:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```
    - Access `http://localhost:8080/message` and confirm the updated message is shown.

---

### **Optional Exercise**

1. **Add environment-specific configuration.**
   - In your Git repo, create `application-dev.properties` and `application-prod.properties` with different `message` values.
   - In `ConfigService`’s `application.properties`, set:
     ```properties
     spring.profiles.active=dev
     ```
   - Verify different `message` values for `dev` or `prod` by switching profiles.

---

## **Conclusion**
You have successfully:
- Created a microservice (`ConfigService`) that initially used a local property.
- Set up a Git-based `ConfigServer`.
- Pulled configurations dynamically using **Spring Cloud Config** with **Spring Boot 3.4.1**.
- Optionally, refreshed configurations at runtime and handled environment-specific setups.

Enjoy centralized configuration management for your microservices! 

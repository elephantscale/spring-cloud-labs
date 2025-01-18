# **Lab 1: Centralized Configuration Management with Spring Cloud Config**

## **Objective**
Set up centralized configuration management for a microservices-based architecture using Spring Cloud Config. Learn how to create a Config Server, integrate it with Git for configuration storage, connect a client microservice to retrieve configuration dynamically, and manage environment-specific properties.

---

## **Lab Steps**

### **Part 1: Create a Client Microservice**

1. **Generate a new Spring Boot project for `ConfigService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: `3.4.1`
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-service`
     - **Name**: `ConfigService`
     - **Package Name**: `com.microservices.configservice`
     - **Dependencies**:
       - Spring Web
   - Extract the zip file into a folder named `ConfigService`.

2. **Import the project into your IDE.**
   - Open your preferred IDE (e.g., IntelliJ, Eclipse, or VS Code).
   - Import the `ConfigService` project as a Maven project.

3. **Add a sample configuration in `application.properties`.**
   - Open `src/main/resources/application.properties` and add:
     ```properties
     message=HelloWorld
     ```

4. **Add a controller to retrieve configuration values.**
   - Create `HelloController.java` in `src/main/java/com/microservices/configservice`:
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

5. **Run and test the `ConfigService` application.**
   - Start the application using:
     ```bash
     ./mvnw spring-boot:run
     ```
   - Access `http://localhost:8080/message` to verify the configuration.

---

### **Part 2: Setting Up the Spring Cloud Config Server**

6. **Set up a Git repository for configuration storage.**
   - Navigate to [GitHub](https://github.com/) and create a new repository named `config-repo`.
   - Clone the repository locally:
     ```bash
     git clone https://github.com/<your-username>/config-repo.git
     cd config-repo
     ```

7. **Add configuration files to the repository.**
   - Inside the cloned repository, create the following files:
     - `application.properties`:
       ```properties
       message=Hello from Config Server!
       ```
     - `config-service.properties`:
       ```properties
       message=Hello from Config Service!
       ```

8. **Commit and push the changes.**
   - Run the following commands:
     ```bash
     git add .
     git commit -m "Added configuration files"
     git push origin main
     ```

9. **Generate a new Spring Boot project for `ConfigServer`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: `3.4.1`
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-server`
     - **Name**: `ConfigServer`
     - **Package Name**: `com.microservices.configserver`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Config Server
   - Extract the zip file into a folder named `ConfigServer`.

10. **Import the project into your IDE.**
    - Import the `ConfigServer` project as a Maven project.

11. **Enable the Config Server.**
    - Open `ConfigServerApplication.java` in `src/main/java/com/microservices/configserver` and add:
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

12. **Configure the Config Server to use the Git repository.**
    - Open `application.properties` in `src/main/resources` and add:
      ```properties
      server.port=8888
      spring.cloud.config.server.git.uri=https://github.com/<your-username>/config-repo
      ```

13. **Run and test the Config Server.**
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```
    - Test the endpoints:
      - Default configuration: `http://localhost:8888/application/default`
      - Service-specific configuration: `http://localhost:8888/config-service/default`

---

### **Part 3: Connect `ConfigService` to `ConfigServer`**

14. **Add Spring Cloud Config dependency to `ConfigService`.**
    - Add the following to `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-config</artifactId>
      </dependency>
      ```

15. **Configure `ConfigService` to use `ConfigServer`.**
    - Update `application.properties`:
      ```properties
      spring.application.name=config-service
      spring.cloud.import=configserver:http://localhost:8888
      ```

16. **Restart the `ConfigService` application.**

17. **Test the dynamic configuration.**
    - Access `http://localhost:8080/message` and verify that the message is retrieved from `ConfigServer`.

---

### **Optional Exercise: Advanced Configuration Management**

1. **Add environment-specific configurations.**
   - Add the following files to the Git repository:
     - `application-dev.properties`:
       ```properties
       message=Hello from the Dev environment!
       ```
     - `application-prod.properties`:
       ```properties
       message=Hello from the Production environment!
       ```

2. **Set the active profile in `ConfigService`.**
   - Update `application.properties`:
     ```properties
     spring.profiles.active=dev
     ```

3. **Test profile-specific configurations.**
   - Restart `ConfigService` and verify the configuration at `http://localhost:8080/message`.

4. **Refresh configuration without restarting.**
   - Add Spring Actuator to `ConfigService`:
     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
     ```
   - Expose Actuator's refresh endpoint:
     ```properties
     management.endpoints.web.exposure.include=refresh
     ```
   - Test dynamic refresh:
     ```bash
     curl -X POST http://localhost:8080/actuator/refresh
     ```

---

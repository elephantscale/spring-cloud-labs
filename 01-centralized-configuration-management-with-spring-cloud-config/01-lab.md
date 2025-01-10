# **Lab 1: Centralized Configuration Management with Spring Cloud Config**

## **Objective**
Set up centralized configuration management for a microservices-based architecture using Spring Cloud Config. Learn to create a Config Server, integrate it with Git for configuration storage, connect a client microservice to retrieve configuration dynamically, and manage environment-specific properties.

---

## **Lab Steps**

### **Part 1: Setting Up the Spring Cloud Config Server**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: Select **3.4.1** from the dropdown menu.
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `config-server`
     - **Name**: `ConfigServer`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Config Server
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `ConfigServer`.

2. **Import the project into your IDE.**
   - Open your favorite Java IDE (e.g., IntelliJ, Eclipse, or VS Code).
   - Import the `ConfigServer` project as a Maven project.

3. **Enable the Config Server in the application.**
   - Open the `ConfigServerApplication.java` file in the `src/main/java/com/microservices/configserver` folder.
   - Add the `@EnableConfigServer` annotation to enable Config Server functionality:
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

4. **Set up the Git repository for configuration storage.**
   - Navigate to [GitHub](https://github.com/) and log in or create an account.
   - Click the **+** button (top-right corner) and select **New repository**.
   - Configure the repository:
     - **Repository name**: `config-repo`
     - **Visibility**: Public (or private, if preferred).
   - Click **Create repository**.

5. **Clone the repository locally.**
   - Open a terminal and run:
     ```bash
     git clone https://github.com/<your-username>/config-repo.git
     cd config-repo
     ```

6. **Add configuration files to the repository.**
   - Inside the cloned repository folder, create the following files:
     - `application.yml`:
       ```yaml
       message: "Hello from Config Server!"
       ```
     - `sample-service.yml`:
       ```yaml
       message: "Hello from Sample Service!"
       ```

7. **Commit and push the changes.**
   - Run:
     ```bash
     git add .
     git commit -m "Added configuration files"
     git push origin main
     ```

8. **Configure the Config Server to use the Git repository.**
   - Open the `application.yml` file in the `ConfigServer` project and add:
     ```yaml
     server:
       port: 8888

     spring:
       cloud:
         config:
           server:
             git:
               uri: https://github.com/<your-username>/config-repo
     ```

9. **Run the Config Server.**
   - Run the `ConfigServerApplication` class from your IDE.

10. **Test the Config Server.**
    - Open a browser and test:
      - Default configuration: `http://localhost:8888/application/default`
      - Service-specific configuration: `http://localhost:8888/sample-service/default`

---

### **Part 2: Creating a Client Microservice**

11. **Generate a new Spring Boot project using Spring Initializr.**
    - Configure the project:
      - **Spring Boot Version**: Select **3.4.1** from the dropdown menu.
      - **Group Id**: `com.microservices`
      - **Artifact Id**: `sample-service`
      - **Name**: `SampleService`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Config Client
    - Extract the project into a folder named `SampleService`.

12. **Import the project into your IDE.**

13. **Configure the client to use the Config Server.**
    - Open the `application.yml` file in `SampleService` and add:
      ```yaml
      spring:
        application:
          name: sample-service
        cloud:
          import: "configserver:http://localhost:8888"
      ```

14. **Add a controller to verify configuration retrieval.**
    - Create `HelloController.java` in `src/main/java/com/microservices/sampleservice`:
      ```java
      package com.microservices.sampleservice;

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

15. **Run the client application.**
    - Start the `SampleService` application.

16. **Test the configuration retrieval.**
    - Access: `http://localhost:8080/message`.

---

### **Part 3: Advanced Configuration Management**

17. **Add Spring Actuator dependency.**
    - Add to `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

18. **Enable Actuator endpoints.**
    - Add to `application.yml`:
      ```yaml
      management:
        endpoints:
          web:
            exposure:
              include: refresh
      ```

19. **Modify the configuration in Git.**
    - Update `sample-service.yml`:
      ```yaml
      message: "Updated message from Config Server!"
      ```
    - Push the changes.

20. **Refresh the client configuration.**
    - Use:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```

21. **Verify the updated configuration.**
    - Access `http://localhost:8080/message`.

---

### **Optional Exercise**

1. **Add environment-specific configurations.**
   - Create in Git:
     - `application-dev.yml`:
       ```yaml
       message: "Hello from the Dev environment!"
       ```
     - `application-prod.yml`:
       ```yaml
       message: "Hello from the Production environment!"
       ```
   - Push changes.

2. **Set a profile.**
   - Add to `application.yml` in `SampleService`:
     ```yaml
     spring:
       profiles:
         active: dev
     ```

3. **Test profile-specific configurations.**
   - Restart the `SampleService` application and verify the message is fetched from the `dev` profile.

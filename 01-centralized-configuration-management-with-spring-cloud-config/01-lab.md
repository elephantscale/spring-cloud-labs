# **Lab 1: Centralized Configuration Management with Spring Cloud Config**

## **Objective**
Set up centralized configuration management for a microservices-based architecture using Spring Cloud Config. Learn to create a Config Server, integrate it with Git for configuration storage, connect a client microservice to retrieve configuration dynamically, and manage environment-specific properties.

---

## **Lab Steps**

### **Part 1: Setting Up the Spring Cloud Config Server**

1. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
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
   - Use the following Git command to clone the repository to your local machine:
     ```bash
     git clone https://github.com/<your-username>/config-repo.git
     cd config-repo
     ```

6. **Add configuration files to the repository.**
   - Inside the cloned repository folder, create the following files:
     - `application.yml`: Default configuration for all services.
       ```yaml
       message: "Hello from Config Server!"
       ```
     - `sample-service.yml`: Configuration specific to the `sample-service`.
       ```yaml
       message: "Hello from Sample Service!"
       ```

7. **Commit and push the changes.**
   - Use the following commands to add, commit, and push the changes:
     ```bash
     git add .
     git commit -m "Added configuration files"
     git push origin main
     ```

8. **Configure the Config Server to use the Git repository.**
   - Open the `application.properties` file in the `ConfigServer` project and add:
     ```properties
     server.port=8888
     spring.cloud.config.server.git.uri=https://github.com/<your-username>/config-repo
     ```

9. **Run the Config Server.**
   - In your IDE, run the `ConfigServerApplication` class.
   - Ensure the server starts on port `8888`.

10. **Test the Config Server.**
    - Open a browser and test configuration retrieval by accessing:
      - Default configuration: `http://localhost:8888/application/default`
      - Service-specific configuration: `http://localhost:8888/sample-service/default`
    - Verify that the responses match the configurations in your Git repository.

---

### **Part 2: Creating a Client Microservice**

11. **Generate a new Spring Boot project using Spring Initializr.**
    - Visit [https://start.spring.io/](https://start.spring.io/).
    - Configure the project:
      - **Group Id**: `com.microservices`
      - **Artifact Id**: `sample-service`
      - **Name**: `SampleService`
      - **Dependencies**:
        - Spring Web
        - Spring Cloud Config Client
    - Click **Generate** to download the project zip file.
    - Extract the downloaded zip file into a folder named `SampleService`.

12. **Import the project into your IDE.**
    - Open your IDE and import the `SampleService` project as a Maven project.

13. **Configure the client to use the Config Server.**
    - Open the `application.properties` file in the `SampleService` project and add:
      ```properties
      spring.application.name=sample-service
      spring.cloud.config.uri=http://localhost:8888
      ```

14. **Add a controller to verify configuration retrieval.**
    - Create a new file `HelloController.java` in the `src/main/java/com/microservices/sampleservice` folder:
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
    - *Explanation*: This controller reads the `message` property from the Config Server and returns it via a REST endpoint.

15. **Run the client application.**
    - Start the `SampleService` application from your IDE.
    - Ensure it starts on the default port `8080`.

16. **Test the configuration retrieval.**
    - Open a browser and access: `http://localhost:8080/message`.
    - Verify that the response matches the `message` property defined in the `sample-service.yml` file in the Git repository.

---

### **Part 3: Advanced Configuration Management**

17. **Add Spring Actuator dependency to the client.**
    - Open the `pom.xml` file and add the following dependency:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

18. **Enable Actuator endpoints in the client.**
    - Add the following to the `application.properties` file in the `SampleService` project:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

19. **Modify the configuration file in the Git repository.**
    - Update the `sample-service.yml` file in the Git repository with a new message:
      ```yaml
      message: "Updated message from Config Server!"
      ```
    - Commit and push the changes:
      ```bash
      git add .
      git commit -m "Updated configuration for sample-service"
      git push origin main
      ```

20. **Refresh the client configuration dynamically.**
    - Use the following command to refresh the configuration without restarting the client:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```

21. **Verify the updated configuration.**
    - Re-access `http://localhost:8080/message` in the browser and ensure the message reflects the updated value from the Git repository.

---

### **Optional Exercise (20 mins)**

1. **Add environment-specific configurations.**
   - Create the following files in the Git repository:
     - `application-dev.yml`: Configuration for the `dev` environment.
       ```yaml
       message: "Hello from the Dev environment!"
       ```
     - `application-prod.yml`: Configuration for the `prod` environment.
       ```yaml
       message: "Hello from the Production environment!"
       ```
   - Commit and push the changes.

2. **Update the client to use a profile.**
   - Add the following to the `application.properties` file in `SampleService`:
     ```properties
     spring.profiles.active=dev
     ```

3. **Test profile-specific configurations.**
   - Restart the `SampleService` application and verify the message is fetched from the `dev` profile.

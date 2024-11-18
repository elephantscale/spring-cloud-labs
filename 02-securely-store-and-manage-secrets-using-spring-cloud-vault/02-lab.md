# **Lab 2: Securely Store and Manage Secrets Using Spring Cloud Vault**

## **Objective**
Learn how to securely store and dynamically reload secrets using Spring Cloud Vault. Configure a microservice to retrieve sensitive configuration data, such as database credentials, from HashiCorp Vault.

---

## **Lab Steps**

### **Part 1: Setting Up HashiCorp Vault**

1. **Download and install HashiCorp Vault.**
   - Visit [https://www.vaultproject.io/downloads](https://www.vaultproject.io/downloads).
   - Download the appropriate binary for your operating system, extract it, and add it to your system's PATH.

2. **Start the Vault development server.**
   - Run the following command in your terminal:
     ```bash
     vault server -dev
     ```
   - Vault will start in development mode on `http://127.0.0.1:8200`.
   - Note the `Root Token` displayed in the logs, e.g., `hvs.XXXX`.

3. **Set up Vault environment variables.**
   - Export the following environment variables to configure the Vault CLI:
     ```bash
     export VAULT_ADDR='http://127.0.0.1:8200'
     export VAULT_TOKEN='hvs.XXXX'  # Replace with your actual Root Token
     ```

4. **Enable the KV secrets engine in Vault.**
   - Run the following command to enable the key-value secrets engine at the `secret/` path:
     ```bash
     vault secrets enable -path=secret kv
     ```

5. **Add secrets to Vault.**
   - Store a key-value pair (`username` and `password`) in Vault for the `user-service`:
     ```bash
     vault kv put secret/user-service username=root password=root123
     ```

6. **Verify the stored secret.**
   - Fetch the secret using the following command:
     ```bash
     vault kv get secret/user-service
     ```
   - Confirm that the secret values (`username` and `password`) are displayed.

---

### **Part 2: Setting Up the Spring Boot Application**

7. **Generate a new Spring Boot project using Spring Initializr.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Vault Config
       - Spring Boot Actuator
   - Click **Generate** to download the project zip file.
   - Extract the downloaded zip file into a folder named `UserService`.

8. **Import the project into your IDE.**
   - Open your favorite IDE (e.g., IntelliJ, Eclipse, or VS Code).
   - Import the `UserService` project as a Maven project.
   - Ensure that all dependencies are downloaded successfully.

9. **Add Spring Cloud Vault configurations.**
   - Open the `src/main/resources/application.properties` file and add the following:
     ```properties
     spring.application.name=user-service
     spring.cloud.vault.uri=http://127.0.0.1:8200
     spring.cloud.vault.token=hvs.XXXX  # Replace with your actual Root Token
     spring.cloud.vault.kv.enabled=true
     spring.cloud.vault.kv.backend=secret
     spring.cloud.vault.kv.application-name=user-service
     ```

10. **Create a configuration class to map secrets.**
    - Create a new file `VaultConfig.java` in the `src/main/java/com/microservices/userservice` folder:
      ```java
      package com.microservices.userservice;

      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.context.annotation.Configuration;

      @Configuration
      @ConfigurationProperties(prefix = "vault")
      public class VaultConfig {

          private String username;
          private String password;

          public String getUsername() {
              return username;
          }

          public void setUsername(String username) {
              this.username = username;
          }

          public String getPassword() {
              return password;
          }

          public void setPassword(String password) {
              this.password = password;
          }
      }
      ```

11. **Inject the VaultConfig class into a REST controller.**
    - Create a new file `VaultController.java` in the `src/main/java/com/microservices/userservice` folder:
      ```java
      package com.microservices.userservice;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class VaultController {

          @Autowired
          private VaultConfig vaultConfig;

          @GetMapping("/secrets")
          public String getSecrets() {
              return "Username: " + vaultConfig.getUsername() + ", Password: " + vaultConfig.getPassword();
          }
      }
      ```

12. **Run the application.**
    - Start the `UserService` application using:
      ```bash
      ./mvnw spring-boot:run
      ```

13. **Test the `/secrets` endpoint.**
    - Open a browser or use Postman to access:
      ```
      http://localhost:8080/secrets
      ```
    - Confirm that the response displays the secrets stored in Vault.

---

### **Part 3: Adding Dynamic Updates for Secrets**

14. **Add Spring Boot Actuator for dynamic refresh.**
    - Open the `pom.xml` file and add:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

15. **Expose the refresh endpoint in Actuator.**
    - Add the following to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

16. **Modify secrets in Vault.**
    - Update the secret stored in Vault:
      ```bash
      vault kv put secret/user-service username=admin password=admin123
      ```

17. **Trigger a refresh of the application’s configuration.**
    - Use the following `curl` command to refresh the application’s configuration without restarting:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```

18. **Verify the updated secrets.**
    - Access the `/secrets` endpoint again:
      ```
      http://localhost:8080/secrets
      ```
    - Confirm that the updated username (`admin`) and password (`admin123`) are displayed.

---

### **Part 4: Testing Environment Profiles**

19. **Set up environment-specific secrets in Vault.**
    - Store secrets for the `dev` environment:
      ```bash
      vault kv put secret/user-service/dev username=devuser password=devpass
      ```

20. **Modify the application to use the `dev` profile.**
    - Add the following to `application.properties`:
      ```properties
      spring.profiles.active=dev
      ```
    - Restart the application and verify that secrets from `secret/user-service/dev` are used.

---

### **Optional Exercises (20 mins)**

1. **Integrate Vault with database properties.**
   - Replace database credentials in your application with Vault-managed secrets:
     - Add the following to `application.properties`:
       ```properties
       spring.datasource.url=jdbc:mysql://localhost:3306/userdb
       spring.datasource.username=${vault.username}
       spring.datasource.password=${vault.password}
       ```
   - Test the application to ensure it works seamlessly with database connections using Vault-managed credentials.

2. **Test Vault with production secrets.**
   - Add secrets for a `prod` environment and switch between profiles (`dev`, `prod`) dynamically.

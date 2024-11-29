# **Lab 2: Securely Store and Manage Secrets Using Spring Cloud Vault**

## **Objective**
Learn how to securely store and dynamically reload secrets using Spring Cloud Vault. Configure a microservice to retrieve sensitive configuration data, such as database credentials, from HashiCorp Vault.

---

## **Lab Steps**

### **Part 1: Installing and Running HashiCorp Vault on Windows**

1. **Download the Vault binary.**
   - Visit [https://www.vaultproject.io/downloads](https://www.vaultproject.io/downloads) and download the latest Vault binary for Windows.

2. **Extract the Vault binary.**
   - Unzip the downloaded file (e.g., `vault_1.14.1_windows_amd64.zip`) to a folder, such as `C:\HashiCorp\Vault`.

3. **Add Vault to the system PATH.**
   - Open **Environment Variables**:
     1. Press `Windows + S` and search for **Environment Variables**.
     2. Click on **Edit the system environment variables**.
     3. Under **System Variables**, select **Path** and click **Edit**.
     4. Add the folder path where the `vault.exe` is located (e.g., `C:\HashiCorp\Vault`).
   - Verify the setup by opening a Command Prompt and running:
     ```cmd
     vault --version
     ```

4. **Start Vault in development mode.**
   - Open a Command Prompt and run:
     ```cmd
     vault server -dev
     ```
   - Vault will start in development mode and display a `Root Token`. Copy the token, as you will need it for configuration.

5. **Set Vault environment variables.**
   - Open a new Command Prompt and set the Vault environment variables:
     ```cmd
     set VAULT_ADDR=http://127.0.0.1:8200
     set VAULT_TOKEN=<your-root-token>
     ```
   - To make these variables persistent, add them to **Environment Variables**.

6. **Enable the KV secrets engine in Vault.**
   - Run the following command in the Command Prompt:
     ```cmd
     vault secrets enable -path=secret kv
     ```

7. **Add secrets to Vault.**
   - Store a key-value pair (`username` and `password`) in Vault for the `user-service`:
     ```cmd
     vault kv put secret/user-service username=root password=root123
     ```

8. **Verify the stored secret.**
   - Fetch the secret using the following command:
     ```cmd
     vault kv get secret/user-service
     ```
   - Confirm that the secret values (`username` and `password`) are displayed.

---

### **Part 2: Setting Up the Spring Boot Application**

9. **Generate a new Spring Boot project using Spring Initializr.**
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

10. **Import the project into your IDE.**
    - Open your favorite IDE (e.g., IntelliJ, Eclipse, or VS Code).
    - Import the `UserService` project as a Maven project.
    - Ensure that all dependencies are downloaded successfully.

11. **Add Spring Cloud Vault configurations.**
    - Open the `src/main/resources/application.properties` file and add the following:
      ```properties
      spring.application.name=user-service
      spring.cloud.vault.uri=http://127.0.0.1:8200
      spring.cloud.vault.token=<your-root-token>
      spring.cloud.vault.kv.enabled=true
      spring.cloud.vault.kv.backend=secret
      spring.cloud.vault.kv.application-name=user-service
      ```

12. **Create a configuration class to map secrets.**
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

13. **Inject the VaultConfig class into a REST controller.**
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

14. **Run the application.**
    - Start the `UserService` application using:
      ```cmd
      mvnw spring-boot:run
      ```

15. **Test the `/secrets` endpoint.**
    - Open a browser or use Postman to access:
      ```
      http://localhost:8080/secrets
      ```
    - Confirm that the response displays the secrets stored in Vault.

---

### **Part 3: Adding Dynamic Updates for Secrets**

16. **Add Spring Boot Actuator for dynamic refresh.**
    - Open the `pom.xml` file and add:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose the refresh endpoint in Actuator.**
    - Add the following to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

18. **Modify secrets in Vault.**
    - Update the secret stored in Vault:
      ```cmd
      vault kv put secret/user-service username=admin password=admin123
      ```

19. **Trigger a refresh of the application’s configuration.**
    - Use the following `curl` command to refresh the application’s configuration without restarting:
      ```cmd
      curl -X POST http://localhost:8080/actuator/refresh
      ```

20. **Verify the updated secrets.**
    - Access the `/secrets` endpoint again:
      ```
      http://localhost:8080/secrets
      ```
    - Confirm that the updated username (`admin`) and password (`admin123`) are displayed.

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

2. **Test Vault with environment profiles.**
   - Add secrets for a `prod` environment and switch between profiles (`dev`, `prod`) dynamically.

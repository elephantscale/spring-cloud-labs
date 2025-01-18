# **Lab 2: Securely Store and Manage Secrets Using Spring Cloud Vault**

## **Objective**
Learn how to securely store and dynamically reload secrets using Spring Cloud Vault. Configure a microservice to retrieve sensitive configuration data, such as database credentials, from HashiCorp Vault.

---

## **Lab Steps**

### **Part 1: Installing and Running HashiCorp Vault**

1. **Download the Vault binary.**
   - Visit [https://www.vaultproject.io/downloads](https://www.vaultproject.io/downloads) and download the latest Vault binary for your operating system.

2. **Extract the Vault binary.**
   - **Windows**:
     - Unzip the downloaded file (e.g., `vault_1.14.1_windows_amd64.zip`) to a folder, such as `C:\HashiCorp\Vault`.
   - **macOS/Linux**:
     - Extract the file:
       ```bash
       unzip vault_<version>_<platform>.zip -d ~/HashiCorp/Vault
       ```

3. **Add Vault to the system PATH.**
   - **Windows**:
     - Add the folder containing `vault.exe` to the PATH via Environment Variables.
   - **macOS/Linux**:
     - Add Vault to the PATH:
       ```bash
       export PATH=$PATH:~/HashiCorp/Vault
       ```

4. **Start Vault in development mode.**
   - Run:
     ```bash
     vault server -dev
     ```
   - Copy the displayed `Root Token` for later use.

5. **Set Vault environment variables.**
   - Configure Vault settings:
     - **Windows** (Command Prompt):
       ```cmd
       set VAULT_ADDR=http://127.0.0.1:8200
       set VAULT_TOKEN=<your-root-token>
       ```
     - **macOS/Linux**:
       ```bash
       export VAULT_ADDR=http://127.0.0.1:8200
       export VAULT_TOKEN=<your-root-token>
       ```

6. **Enable the KV secrets engine in Vault.**
   - Run:
     ```bash
     vault secrets enable -path=secret kv
     ```

7. **Add secrets to Vault.**
   - Store secrets for the `user-service`:
     ```bash
     vault kv put secret/user-service username=root password=root123
     ```

8. **Verify the stored secret.**
   - Fetch the secret:
     ```bash
     vault kv get secret/user-service
     ```

---

### **Part 2: Setting Up the Spring Boot Application**

9. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure the project:
     - **Spring Boot Version**: `3.4.1`
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Package Name**: `com.microservices.userservice`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Vault Config
       - Spring Boot Actuator
   - Extract the downloaded zip file into a folder named `UserService`.

10. **Import the project into your IDE.**
    - Open the project in your preferred IDE.

11. **Add Spring Cloud Vault configurations.**
    - Open `src/main/resources/application.properties` and add:
      ```properties
      spring.application.name=user-service
      spring.cloud.vault.uri=http://127.0.0.1:8200
      spring.cloud.vault.token=<your-root-token>
      spring.cloud.vault.kv.enabled=true
      spring.cloud.vault.kv.backend=secret
      spring.cloud.vault.kv.application-name=user-service
      ```

12. **Create a configuration class to map secrets.**
    - Create `VaultConfig.java` in `src/main/java/com/microservices/userservice`:
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

13. **Create a REST controller to expose secrets.**
    - Create `VaultController.java`:
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
    - Start the application:
      ```bash
      ./mvnw spring-boot:run
      ```

15. **Test the `/secrets` endpoint.**
    - Access:
      ```
      http://localhost:8080/secrets
      ```

---

### **Part 3: Adding Dynamic Updates for Secrets**

16. **Add Actuator for dynamic refresh.**
    - Add to `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose the refresh endpoint.**
    - Add to `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

18. **Update secrets in Vault.**
    - Modify the stored secrets:
      ```bash
      vault kv put secret/user-service username=admin password=admin123
      ```

19. **Trigger a configuration refresh.**
    - Use the following command:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```

20. **Verify the updated secrets.**
    - Access `http://localhost:8080/secrets` and confirm the updated values.

---

### **Optional Exercises**

1. **Use Vault for database credentials.**
   - Update the `application.properties` to use Vault-managed secrets for database credentials:
     ```properties
     spring.datasource.url=jdbc:mysql://localhost:3306/userdb
     spring.datasource.username=${vault.username}
     spring.datasource.password=${vault.password}
     ```

2. **Integrate environment profiles.**
   - Add separate secrets for `dev` and `prod` environments and test switching between them dynamically.

# **Lab 2: Securely Store and Manage Secrets Using Spring Cloud Vault (Spring Boot 3.4.1)**

## **Objective**
Learn how to securely store and dynamically reload secrets using **Spring Cloud Vault**. Configure a **Spring Boot 3.4.1** microservice to retrieve sensitive configuration data (e.g., database credentials) from **HashiCorp Vault**, and refresh them at runtime without restarting the application.

---

## **Lab Steps**

### **Part 1: Installing and Running HashiCorp Vault**

1. **Download the Vault binary.**
   - Visit [Vault Downloads](https://www.vaultproject.io/downloads) and download the latest Vault binary for your operating system.

2. **Extract the Vault binary.**
   - **Windows**:
     - Unzip (e.g., `vault_<version>_windows_amd64.zip`) to a folder like `C:\HashiCorp\Vault`.
   - **macOS/Linux**:
     - Extract with:
       ```bash
       unzip vault_<version>_<platform>.zip -d ~/HashiCorp/Vault
       ```

3. **Add Vault to the system PATH.**
   - **Windows**:
     - Add the folder with `vault.exe` to **Path** via Environment Variables.
   - **macOS/Linux**:
     - Update your shell config (e.g., `~/.bashrc` or `~/.zshrc`):
       ```bash
       export PATH=$PATH:~/HashiCorp/Vault
       ```

4. **Start Vault in development mode.**
   - From a terminal:
     ```bash
     vault server -dev
     ```
   - Copy the displayed **Root Token** (e.g., `root` or `s.xxxxx...`).

5. **Set Vault environment variables.**
   - **Windows (Command Prompt)**:
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
   - This creates a secret engine at `secret/`.

7. **Add secrets to Vault.**
   - For example, store user-service credentials:
     ```bash
     vault kv put secret/user-service username=root password=root123
     ```
   - This creates a secret at `secret/user-service` with two key-value pairs:
     - `username=root`
     - `password=root123`

8. **Verify the stored secret.**
   - Run:
     ```bash
     vault kv get secret/user-service
     ```
   - Confirm the displayed data matches what you stored.

---

### **Part 2: Setting Up the Spring Boot Application (UserService)**

9. **Generate a new Spring Boot project for `UserService`.**
   - Visit [https://start.spring.io/](https://start.spring.io/).
   - Configure:
     - **Spring Boot Version**: **3.4.1**
     - **Group Id**: `com.microservices`
     - **Artifact Id**: `user-service`
     - **Name**: `UserService`
     - **Package Name**: `com.microservices.userservice`
     - **Dependencies**:
       - Spring Web
       - Spring Cloud Vault Config
       - Spring Boot Actuator
   - Download and extract into a folder named `UserService`.

10. **Import `UserService` into your IDE.**

11. **Add Spring Cloud Vault configurations.**
    - In `src/main/resources/application.properties`:
      ```properties
      spring.application.name=user-service

      # Vault connection
      spring.cloud.vault.uri=http://127.0.0.1:8200
      spring.cloud.vault.token=<your-root-token>
      spring.cloud.vault.kv.enabled=true
      spring.cloud.vault.kv.backend=secret
      spring.cloud.vault.kv.application-name=user-service
      ```

12. **Create a configuration class to map secrets.**
    - In `src/main/java/com/microservices/userservice`, create `VaultConfig.java`:
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

13. **Expose secrets via a REST controller.**
    - In `VaultController.java`:
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
              return "Username: " + vaultConfig.getUsername() + 
                     ", Password: " + vaultConfig.getPassword();
          }
      }
      ```

14. **Run the application.**
    - From the `UserService` directory:
      ```bash
      ./mvnw spring-boot:run
      ```

15. **Test the `/secrets` endpoint.**
    - Go to:
      ```
      http://localhost:8080/secrets
      ```
    - Confirm it returns **Username: root, Password: root123** (from Vault).

---

### **Part 3: Enabling Dynamic Updates**

16. **Add Actuator for dynamic refresh.**
    - In `pom.xml`:
      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```

17. **Expose the refresh endpoint.**
    - In `application.properties`:
      ```properties
      management.endpoints.web.exposure.include=refresh
      ```

18. **Update secrets in Vault.**
    - For example:
      ```bash
      vault kv put secret/user-service username=admin password=admin123
      ```

19. **Trigger a configuration refresh.**
    - Use:
      ```bash
      curl -X POST http://localhost:8080/actuator/refresh
      ```

20. **Verify the updated secrets.**
    - Re-check `http://localhost:8080/secrets`.
    - Confirm **Username: admin, Password: admin123** is displayed.

---

## **Conclusion**
In this lab, you successfully:
- **Installed and started Vault** in development mode.
- **Stored secrets** (username/password) in Vault under `secret/user-service`.
- **Configured a Spring Boot 3.4.1 microservice** (`UserService`) to fetch and map those secrets dynamically using **Spring Cloud Vault**.
- **Enabled Actuator** to refresh secrets at runtime without restarting the application.

This approach ensures secure management of sensitive information (like credentials) and makes secret rotation and updating seamless.
